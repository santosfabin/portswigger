# Lab: Unprotected admin functionality with unpredictable URL

## Enunciado (traduzido)
Este lab tem um painel de admin desprotegido. Ele está localizado em um endereço imprevisível, mas esse endereço é divulgado em algum lugar da aplicação.
Resolva o lab acessando o painel de admin e deletando o usuário carlos.

## O que fiz

Como o painel de admin não tem nenhum botão ou link visível na tela, comecei inspecionando o código-fonte da página inicial para ver se havia alguma pista deixada no front-end.

Analisando o HTML e os scripts carregados na página, encontrei uma referência a uma rota administrativa:

![image](../../imgs/02/02/1.png)

A rota encontrada foi `/admin-4g0nar`.

Ao acessar esse caminho diretamente no navegador, o painel abriu sem exigir autenticação.

Dentro dele, utilizei a opção de deletar o usuário `carlos`, finalizando o lab:

![image](../../imgs/02/02/2.png)

---
[⬅ Voltar](../../../README.md)