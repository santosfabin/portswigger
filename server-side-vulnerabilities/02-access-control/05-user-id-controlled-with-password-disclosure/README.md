# Lab: User ID controlled by request parameter with password disclosure

## Enunciado (traduzido)
Este lab tem uma página de conta do usuário que contém a senha atual do usuário, pré-preenchida em um input mascarado.

Para resolver o lab, recupere a senha do administrador, depois use-a para deletar o usuário carlos.

Você pode logar na sua própria conta usando as credenciais: `wiener:peter`

## O que fiz

Iniciei fazendo login afim de ver possíveis lugares onde podemos testar.

Após fazer login, olhando na url, temos o usuário que está logado.

Vamos testar trocar nosso user pelo `administrator` para ver o que acontece.

Ele tratou como se fosse o próprio user, então basta ver a senha e fazer login com ele. Para isso, você pode ir no inspecionar e trocar de `type="password"` para `type="text"`.

![image](../../imgs/02/05/1.png)

Então é só logar colocando a senha que descobrimos e deletar o user carlos.

![image](../../imgs/02/05/2.png)

---
[⬅ Voltar](../../README.md)