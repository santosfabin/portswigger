# Lab: SQL injection vulnerability allowing login bypass

## Enunciado (traduzido)
Este lab contém uma vulnerabilidade de SQL injection na função de login.

Para resolver o lab, realize um ataque de SQL injection que faça login na aplicação como o usuário `administrator`.

## O que fiz

A funcionalidade de login provavelmente executa uma consulta SQL verificando se o par de usuário e senha coincide:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'peter'
```

Para logar como `administrator` sem saber a senha, o objetivo é fazer a query validar apenas o nome de usuário e ignorar a checagem da senha.

No campo **username**, inseri:
```text
administrator'--
```

No campo de senha, preenchi qualquer valor apenas para passar pela validação visual do front-end.

Com a injeção, a query executada no backend virou:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'qualquercoisa'
```

A sequência `--` (com espaço) comenta todo o restante da consulta original a partir dali, anulando a verificação `AND password = '...'`. Como o usuário `administrator` existe no banco, a query retornou o registro com sucesso e a sessão de admin foi iniciada.

---
[⬅ Voltar](../../../README.md)