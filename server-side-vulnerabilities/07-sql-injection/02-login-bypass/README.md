# Lab: SQL injection vulnerability allowing login bypass

## Enunciado (traduzido)

Este lab contém uma vulnerabilidade de SQL injection na função de login.

Para resolver o lab, realize um ataque de SQL injection que faça login na aplicação como o usuário `administrator`.

## O que fiz

Ao navegar pela aplicação, inspecionei o HTML de uma página de produto e encontrei a tag de imagem:

```html
<img src="/image?filename=18.jpg" />
```

O parâmetro `filename` recebe diretamente o nome do arquivo. Isso é imediatamente suspeito: se a aplicação não sanitizar esse valor, posso controlar qual arquivo o servidor vai buscar no disco.

O servidor provavelmente monta o caminho assim internamente:

```
/var/www/images/ + filename
```

Se eu passar `../../../etc/passwd`, o caminho resultante seria:

```
/var/www/images/../../../etc/passwd
```

Que o sistema operacional resolve como:

```
/etc/passwd
```

---

Não sei de antemão onde no sistema de arquivos o servidor armazena as imagens. Precisei testar quantos `../` são necessários para chegar à raiz (`/`) e então descer até `/etc/passwd`.

Testei incrementalmente:

**1 nível — sem efeito:**

```
/image?filename=../etc/passwd
```

→ Erro ou imagem quebrada. Ainda dentro da pasta de imagens ou abaixo dela.

**2 níveis — ainda sem efeito:**

```
/image?filename=../../etc/passwd
```

→ Mesma resposta. Ainda não cheguei à raiz.

**3 níveis — funcionou:**

```
/image?filename=../../../etc/passwd
```

→ O servidor retornou o conteúdo do arquivo `/etc/passwd`.

Isso indica que as imagens ficam 3 níveis abaixo da raiz do sistema de arquivos, provavelmente em algo como `/var/www/images/`.

---

O arquivo `/etc/passwd` é um arquivo padrão de sistemas Unix/Linux que lista os usuários do sistema. Cada linha tem o formato:

```
username:password:UID:GID:info:home:shell
```

O campo `password` hoje geralmente contém apenas `x`, indicando que a senha real fica em `/etc/shadow` (que é mais restrito). Mas o `/etc/passwd` ainda é valioso para um atacante porque revela:

- Quais usuários existem no sistema
- Os diretórios home de cada usuário
- Quais shells estão em uso

A simples leitura desse arquivo já configura uma **divulgação de informação sensível** e prova que o atacante tem leitura arbitrária de arquivos no servidor.

---

[⬅ Voltar](../../../README.md)
