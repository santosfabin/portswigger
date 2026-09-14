# Lab: Web shell upload via Content-Type restriction bypass

## Enunciado (traduzido)
Este lab contém uma função de upload de imagem vulnerável. Ela tenta impedir que usuários façam upload de tipos de arquivo inesperados, mas se baseia na checagem de um input controlável pelo usuário para fazer essa verificação.

Para resolver o lab, faça upload de uma web shell básica em PHP e use-a para exfiltrar o conteúdo do arquivo `/home/carlos/secret`. Envie esse segredo usando o botão fornecido no banner do lab.

Você pode logar na sua própria conta usando as credenciais: `wiener:peter`

## O que fiz

O processo é praticamente idêntico ao lab anterior ([Remote code execution via web shell upload](../01-remote-code-execution-via-web-shell-upload/README.md)): fizemos upload de um arquivo `.php` com o conteúdo:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

A diferença é que aqui o servidor valida o tipo do arquivo checando o header `Content-Type` enviado na requisição, que é um valor controlável por nós — ou seja, não é uma validação real do conteúdo do arquivo.

Interceptando o upload no Burp, o `Content-Type` vinha como `application/x-php` (o que é bloqueado pelo servidor):

![image](../../imgs/05/02/1.png)

Bastou trocar o valor para `image/jpeg`, fazendo o servidor acreditar que era uma imagem legítima:

![image](../../imgs/05/02/2.png)

Depois disso, é só acessar a imagem normalmente pra executar o PHP, como no lab anterior.

---
[⬅ Voltar](../../README.md)