# Lab: SQL injection UNION attack, finding a column containing text

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection no filtro de categoria de produtos. Os resultados da consulta são retornados na resposta da aplicação, portanto, você pode usar um ataque UNION para recuperar dados de outras tabelas. Para construir esse ataque, primeiro você precisa determinar o número de colunas retornadas pela query original — você pode fazer isso usando a técnica aprendida no lab anterior. O próximo passo é identificar qual coluna é compatível com dados do tipo texto (string).

O lab fornecerá um valor aleatório que você precisará fazer aparecer nos resultados da consulta. Para resolver o lab, execute um ataque de SQL injection UNION que retorne uma linha adicional contendo o valor fornecido. Essa técnica ajuda a determinar quais colunas são compatíveis com texto.

## O que fiz

Para conseguirmos extrair dados úteis (como nomes de usuários e senhas) através de um ataque `UNION`, precisamos encontrar colunas que aceitem o tipo de dado **texto (string)** e que sejam exibidas na página web.

O processo é dividido em duas etapas:

---

### 1. Confirmar a quantidade de colunas

Primeiro, precisamos saber quantas colunas a query original possui. Podemos usar o `ORDER BY` ou ir incrementando valores `NULL` no `UNION SELECT`:
```
.../filter?category=Gifts' UNION SELECT NULL, NULL, NULL--
```

A requisição respondeu normalmente com **3 colunas** e sem erros, confirmando o tamanho da consulta.

---

### 2. Testar a compatibilidade de tipo (procurando colunas de texto)

Agora que sabemos que existem 3 colunas, precisamos testar uma a uma substituindo o `NULL` por um caractere de texto (como `'a'`). 

Se a coluna na query original for de um tipo incompatível (por exemplo, um número inteiro ou data), o banco de dados disparará um erro de conversão de tipos:
```
.../filter?category=Gifts' UNION SELECT 'a', NULL, NULL-- (Erro - Coluna 1 não aceita texto)
.../filter?category=Gifts' UNION SELECT NULL, 'a', NULL-- (Sucesso - Coluna 2 aceita texto!)
.../filter?category=Gifts' UNION SELECT NULL, NULL, 'a'-- (Erro - Coluna 3 não aceita texto)
```

Como apenas a **segunda coluna** retornou código de sucesso e exibiu o caractere `'a'` na tela, sabemos que a segunda posição é a única compatível com strings.

---

### 3. Resolução do Lab

O enunciado do lab exige que façamos a aplicação exibir uma string aleatória específica gerada no cabeçalho da página (exemplo: `Make the database retrieve the string: 'abc123xyz'`).

Substituímos o valor de teste `'a'` pela string exata exigida pelo lab na 2ª coluna:
```
.../filter?category=Gifts' UNION SELECT NULL, 'abc123xyz', NULL--
```
Ao enviar essa requisição, o banco retorna a string solicitada na segunda coluna, ela é renderizada na resposta da página e o lab é resolvido com sucesso.

---

[⬅ Voltar](../../../README.md)