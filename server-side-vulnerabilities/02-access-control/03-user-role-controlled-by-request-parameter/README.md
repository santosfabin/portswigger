# Lab: User role controlled by request parameter

## Enunciado (traduzido)
Este lab tem um painel de admin em `/admin`, que identifica administradores usando um cookie forjável.
Resolva o lab acessando o painel de admin e deletando o usuário `carlos`.

Você pode logar na sua própria conta usando as credenciais: `wiener:peter`

## O que fiz

Fiz login com as credenciais fornecidas (`wiener:peter`) e interceptei o tráfego no Burp Suite para analisar como a aplicação gerencia a sessão do usuário.

Na resposta da autenticação, reparei que o servidor definiu um cookie chamado `Admin=false`:

![image](../../imgs/02/03/1.png)

Isso indica que o papel (role) do usuário está sendo controlado diretamente pelo valor desse cookie no lado do cliente.

Podemos explorar isso de duas formas:

1. Alterando o cookie direto nas ferramentas de desenvolvedor do navegador (DevTools) de `Admin=false` para `Admin=true`:

![image](../../imgs/02/03/2.png)

2. Interceptando a requisição no Burp Suite e modificando o cabeçalho `Cookie: Admin=true` antes de enviar:

![image](../../imgs/02/03/3.png)

Com o cookie alterado para `Admin=true`, ao acessar a rota `/admin`, o painel administrativo foi liberado e pude deletar o usuário `carlos`.

---
[⬅ Voltar](../../../README.md)