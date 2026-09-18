# Lab: User ID controlled by request parameter, with unpredictable user IDs

## Enunciado (traduzido)
Este lab tem uma vulnerabilidade de escalação horizontal de privilégios na página de conta do usuário, mas identifica os usuários com GUIDs.

Para resolver o lab, encontre o GUID do carlos, depois envie a API key dele como solução.

Você pode logar na sua própria conta usando as credenciais: `wiener:peter`

## O que fiz

Nesse, basta procurarmos algum usuário, e que nesse caso nos pede pelo user carlos.

Entrando em alguns post, existe esse com essa url `https://….net/post?postId=6`

Procurando pelo inspecionar no seu nome já achamos o id daquele user:

`b7afa08d-2afb-469d-9bd8-e58790574311`

![image](../../imgs/02/04/1.png)

Fazendo login normalmente na nossa conta, é mostrado na url o id que é pego para mostrar as informações da nossa conta.

Então na url `…net/my-account?id=uuid` basta colocar o do carlos que achamos.

Então enviar a API key dele em "Submit solution".

![image](../../imgs/02/04/2.png)

---
[⬅ Voltar](../../../README.md)