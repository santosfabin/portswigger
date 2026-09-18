# Lab: OS command injection, simple case

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de OS command injection no verificador de estoque de produtos.

A aplicação executa um comando shell contendo IDs de produto e loja fornecidos pelo usuário, e retorna a saída bruta do comando na resposta.

Para resolver o lab, execute o comando `whoami` para descobrir o nome do usuário atual.

## O que fiz

Ao testar a funcionalidade de "Check stock" na página de um produto, interceptei a requisição no Burp Suite para analisar os parâmetros enviados ao backend.

A requisição envia dois parâmetros no corpo: `productId` e `storeId`. 

Pelo painel lateral do **Inspector** no Burp, conseguimos ver e editar esses parâmetros de forma decodificada:

![image](../../imgs/06/01/1.png)

Cliquei na seta ao lado do parâmetro `storeId` para abrir a edição direta do valor:

![image](../../imgs/06/01/2.png)

Como a aplicação executa um comando interno no servidor usando esses valores, utilizei o operador `&` para encadear um novo comando. Inseri `1 & whoami`, e o próprio Burp cuidou da codificação para URL encoding (`1%20%26%20whoami`):

![image](../../imgs/06/01/3.png)

Ao aplicar as alterações e enviar a requisição, o servidor executou o comando no terminal e retornou a saída do `whoami` (`peter-...`) no corpo da resposta HTTP, concluindo o laboratório.

---
[⬅ Voltar](../../../README.md)