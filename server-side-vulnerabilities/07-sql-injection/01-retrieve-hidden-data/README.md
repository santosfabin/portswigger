# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no filtro de categoria de produtos. Quando o usuário seleciona uma categoria, a aplicação executa uma query SQL parecida com esta:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Para resolver o lab, realize um ataque de SQL injection que faça a aplicação exibir um ou mais produtos ainda não lançados (`released = 0`).

## O que fiz

Olhando a URL do filtro de categoria (`…/filter?category=Accessories`), dá pra perceber que o valor de `category` provavelmente é concatenado direto na query SQL.

A query original filtra por categoria E por `released = 1`, ou seja, só mostra produtos já lançados.

O objetivo é fazer a condição `WHERE` ser sempre verdadeira, ignorando o filtro de `released`. Pra isso, fechamos a aspa do valor original e injetamos uma condição sempre verdadeira, comentando o resto da query com `-- ` (com espaço no final):

**URL final:**

```
…/filter?category=Accessories'+OR+1=1--
```

Isso faz a query virar:

```sql
SELECT * FROM products WHERE category = 'Accessories' OR 1=1-- ' AND released = 1
```

Como `1=1` é sempre verdadeiro, a condição inteira passa a ser verdadeira pra qualquer linha, e o `-- ` comenta o resto da query (incluindo o `AND released = 1`), retornando todos os produtos, inclusive os não lançados.

---

[⬅ Voltar](../../../README.md)
