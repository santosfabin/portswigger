# Lab: File path traversal, simple case

## Enunciado (traduzido)
Este lab contém uma vulnerabilidade de path traversal na exibição de imagens de produtos.
Para resolver, é preciso recuperar o conteúdo do arquivo `/etc/passwd`.

## O que fiz

Para esse lab precisamos encontrar alguma imagem ou algo que pegue algo do próprio computador.

Na url `https://….net/product?productId=2` existe uma imagem e nela temos isso:

```jsx
<img src="/image?filename=18.jpg">
```

E aí vem a pergunta, por que `filename=18.jpg`?

Então vamos testar o arquivo pedido pelo lab: `/etc/passwd`

Precisamos passar `../../../etc/passwd` no `filename`:

```jsx
/image?filename=../../../etc/passwd
```

---
[⬅ Voltar](../../README.md)