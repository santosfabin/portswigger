# Lab: Basic SSRF against the local server

## Enunciado (traduzido)

Este lab tem uma funcionalidade de verificação de estoque que busca dados de um sistema interno.

Para resolver o lab, altere a URL de verificação de estoque para acessar a interface de admin em `http://localhost/admin` e delete o usuário `carlos`.

## O que fiz

Interceptei a funcionalidade de verificação de estoque de um produto no Burp Suite para analisar a requisição:

![image](../../imgs/04/01/1.png)

No corpo da requisição `POST /product/stock`, identifiquei o parâmetro `stockApi` apontando para uma URL de backend:

![image](../../imgs/04/01/2.png)

Enviei a requisição para o Repeater e alterei o valor do parâmetro `stockApi` para `http://localhost/admin`:

![image](../../imgs/04/01/3.png)

O servidor processou a requisição internamente e retornou o HTML do painel administrativo no corpo da resposta:

![image](../../imgs/04/01/4.png)

![image](../../imgs/04/01/5.png)

![image](../../imgs/04/01/6.png)

Analisando a resposta HTML no Burp, identifiquei a rota utilizada para deletar usuários (`/admin/delete?username=carlos`):

![image](../../imgs/04/01/7.png)

Como o painel só é acessível a partir do próprio servidor local, executei a ação enviando a rota de deleção diretamente pelo parâmetro vulnerável via SSRF:
`stockApi=http://localhost/admin/delete?username=carlos`

O servidor executou a requisição interna e o usuário `carlos` foi deletado.

---
[⬅ Voltar](../../../README.md)