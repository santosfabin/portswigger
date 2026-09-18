# Lab: File path traversal, simple case

## Enunciado (traduzido)
Este lab contém uma vulnerabilidade de path traversal na exibição de imagens de produtos.
Para resolver, é preciso recuperar o conteúdo do arquivo `/etc/passwd`.

## O que fiz

Ao acessar a página de um produto (`/product?productId=2`), inspecionei o carregamento da imagem e encontrei a seguinte chamada:

```html
<img src="/image?filename=18.jpg">
```

O endpoint `/image` recebe o nome do arquivo diretamente pelo parâmetro `filename`. 

Como a aplicação busca a imagem dentro de um diretório interno no servidor, usei a sequência `../../../` para voltar até a raiz do sistema e acessar o `/etc/passwd`:

```http
GET /image?filename=../../../etc/passwd HTTP/1.1
```

O servidor processou o caminho e retornou o conteúdo do `/etc/passwd` na resposta, resolvendo o laboratório.

---
[⬅ Voltar](../../../README.md)