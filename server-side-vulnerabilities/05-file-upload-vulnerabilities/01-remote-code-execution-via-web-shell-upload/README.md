# Lab: Remote code execution via web shell upload

## Enunciado (traduzido)
Este lab contém uma função de upload de imagem vulnerável. Ela não realiza nenhuma validação nos arquivos que os usuários enviam antes de armazená-los no sistema de arquivos do servidor.

Para resolver o lab, faça upload de uma web shell básica em PHP e use-a para exfiltrar o conteúdo do arquivo `/home/carlos/secret`. Envie esse segredo usando o botão fornecido no banner do lab.

Você pode logar na sua própria conta usando as credenciais: `wiener:peter`

## O que fiz

Fiz login com a conta de teste (`wiener:peter`) e acessei a funcionalidade de upload de avatar no perfil da conta.

Como o servidor não valida o tipo nem a extensão dos arquivos enviados, criei um script PHP simples (`exploit.php`) para ler o arquivo do Carlos:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Fiz o upload do arquivo e inspecionei a página para encontrar o caminho onde os avatares ficam salvos:

![image](../../imgs/05/01/1.png)

Acessei diretamente a URL do arquivo no servidor (`/files/avatars/exploit.php`).

O backend interpretou e executou o código PHP, retornando o conteúdo do arquivo `/home/carlos/secret` no corpo da resposta. Copiei o segredo e enviei na solução.

---
[⬅ Voltar](../../../README.md)