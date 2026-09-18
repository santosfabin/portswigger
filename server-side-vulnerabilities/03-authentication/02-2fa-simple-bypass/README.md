# Lab: 2FA simple bypass

## Enunciado (traduzido)
A autenticação de dois fatores deste lab pode ser contornada. Você já obteve um username e senha válidos, mas não tem acesso ao código de verificação 2FA do usuário. Para resolver o lab, acesse a página de conta do Carlos.

Suas credenciais: `wiener:peter`
Credenciais da vítima: `carlos:montoya`

## O que fiz

Fiz login normalmente com as credenciais do Carlos.

Na tela de verificação do 2FA, em vez de inserir o código, voltei para a página inicial (landing page), atualizei e cliquei em `My account`.

Nesse caso, a aplicação já havia salvo a sessão do usuário como logada antes mesmo de terminar a verificação de dois fatores, permitindo acessar a conta sem informar o código.

---
[⬅ Voltar](../../../README.md)