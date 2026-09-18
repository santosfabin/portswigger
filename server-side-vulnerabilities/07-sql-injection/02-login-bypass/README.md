# Lab: SQL injection vulnerability allowing login bypass

## Enunciado (traduzido)
Este lab contém uma vulnerabilidade de SQL injection na função de login.

Para resolver o lab, realize um ataque de SQL injection que faça login na aplicação como o usuário `administrator`.

## O que fiz

O login provavelmente executa uma query parecida com esta, verificando se existe um usuário com aquele username E aquela senha:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'peter'
```

Pra logar como `administrator` sem saber a senha, o objetivo é fazer a query considerar a linha do `administrator` como válida, ignorando a checagem de senha.

No campo **username**, coloquei:
```
administrator ' or 1=1 --
```

E no campo **senha**, qualquer valor, só pra passar pela validação de campo obrigatório (o valor da senha não importa, pois nem chega a ser checado).

Isso faz a query virar:

```sql
SELECT * FROM users WHERE username = 'administrator'-- ' AND password = 'qualquercoisa'
```

O `-- ` comenta todo o resto da query a partir dali, incluindo o `AND password = '...'`. Ou seja, a query passa a checar só se existe um usuário chamado `administrator`, ignorando completamente a senha — e como esse usuário existe, o login é feito com sucesso.

---
[⬅ Voltar](../../../README.md)