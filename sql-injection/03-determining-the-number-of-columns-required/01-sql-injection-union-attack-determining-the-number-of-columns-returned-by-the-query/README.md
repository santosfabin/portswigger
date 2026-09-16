# Lab: SQL injection UNION attack, determining the number of columns returned by the query

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no filtro de categoria de produtos. Os resultados da query são retornados na resposta da aplicação, então você pode usar um ataque UNION para recuperar dados de outras tabelas. O primeiro passo desse tipo de ataque é determinar o número de colunas que estão sendo retornadas pela query. Você usará essa técnica nos labs subsequentes para construir o ataque completo.

Para resolver o lab, determine o número de colunas retornadas pela query realizando um ataque de SQL injection UNION que retorna uma linha adicional contendo valores nulos.

## O que fiz

Primeiro, precisamos encontrar o filtro mencionado pelo desafio. Ele está localizado na seção **"Refine your search"**. Clicando em qualquer categoria, por exemplo `Gifts`, já temos o ponto de injeção na URL.

### 1. Descobrindo a quantidade de colunas

Para descobrir quantas colunas existem na query original, utilizei a cláusula `ORDER BY`. O `ORDER BY` ordena os resultados pelo índice da coluna. Se pedirmos para ordenar por uma coluna que não existe (por exemplo, coluna 4 quando só existem 3), o banco de dados retornará um erro.

Testei diretamente no parâmetro `category` da URL, incrementando de 1 em 1:

```
.../filter?category=Gifts' ORDER BY 1--
.../filter?category=Gifts' ORDER BY 2--
.../filter?category=Gifts' ORDER BY 3--
.../filter?category=Gifts' ORDER BY 4--
```

No número `4` a aplicação retornou um erro interno, logo concluímos que a consulta original possui **exatamente 3 colunas**.

---

### 2. Fundamentos do operador UNION

O comando `UNION` permite anexar os resultados de uma consulta adicional aos resultados da consulta original. No entanto, para que um `UNION` funcione, duas regras obrigatórias devem ser atendidas:

1. **Compatibilidade na quantidade de colunas:** O número de colunas selecionadas no `UNION SELECT` deve ser exatamente igual ao da consulta original.
2. **Compatibilidade no tipo de dado:** O tipo de dado de cada coluna no `UNION` deve ser compatível com o tipo de dado da coluna na mesma posição da query original (ex: número com número, texto com texto).

Se tentarmos uma quantidade incorreta, a query falha:

```
.../filter?category=Gifts' UNION SELECT NULL, NULL--           (Erro: apenas 2 colunas)
.../filter?category=Gifts' UNION SELECT NULL, NULL, NULL, NULL-- (Erro: 4 colunas)
```

> **Por que usar `NULL`?**  
> O valor `NULL` representa a ausência de valor e pode ser convertido para a maioria dos tipos de dados (texto, número, data) na maioria dos bancos (como PostgreSQL, SQL Server e MySQL), servindo como um "coringa" seguro para testar o número de colunas sem se preocupar com os tipos de dados de imediato.

---

### 3. Particularidades de Sintaxe por Banco de Dados

É importante conhecer como diferentes bancos se comportam durante esse ataque:

* **Oracle Database:**
  * No Oracle, todo comando `SELECT` é obrigado a ter a cláusula `FROM` apontando para uma tabela válida.
  * Para consultas que não usam tabelas reais, o Oracle fornece uma tabela padrão do sistema chamada `DUAL`.
  * Em um banco Oracle, o payload precisaria ser:
    ```sql
    ' UNION SELECT NULL, NULL, NULL FROM DUAL--
    ```
* **Comentários de Linha (`--` vs `#`):**
  * O `--` serve para comentar (ignorar) o restante da query original do sistema.
  * No **MySQL**, a sequência `--` precisa obrigatoriamente ser seguida por um espaço (`-- `) para ser interpretada como comentário. Alternativamente, no MySQL você também pode usar a cerquilha/hash (`#`).

---

### 4. Resolução do Lab

Como já descobrimos que a consulta possui 3 colunas, injetamos 3 valores `NULL`:

```
.../filter?category=Gifts' UNION SELECT NULL, NULL, NULL--
```

Isso retorna uma linha adicional preenchida com valores nulos com sucesso, validando a estrutura e resolvendo o lab.

---

[⬅ Voltar](../../README.md)