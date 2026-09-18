# Lab: Basic SSRF against another back-end system

## Enunciado (traduzido)

Este lab tem uma funcionalidade de verificação de estoque que busca dados de um sistema interno.

Para resolver o lab, use a funcionalidade de verificação de estoque para escanear a faixa interna `192.168.0.X` em busca de uma interface de admin na porta `8080`, depois use-a para deletar o usuário `carlos`.

## O que fiz

A mecânica inicial é a mesma do lab anterior, manipulando o parâmetro `stockApi` na requisição de estoque.

Porém, o painel administrativo agora está localizado em algum host dentro da rede interna (`192.168.0.X:8080/admin`). Para descobrir o IP correto, enviei a requisição para o **Intruder** e marquei o último octeto do IP como payload:
`stockApi=http://192.168.0.§1§:8080/admin`

Configurei os payloads para iterar números de 1 a 254 (hosts válidos na sub-rede `/24`, excluindo o `0` de rede e o `255` de broadcast):

![image](../../imgs/04/02/1.png)

Ao ordenar as respostas pelo `Status code`, enquanto a maioria retornou erro `500`, o IP `192.168.0.67` retornou `200 OK`, confirmando a interface administrativa:

![image](../../imgs/04/02/2.png)

Com o IP correto descoberto, enviei a requisição final com o comando de deleção via SSRF:

```
stockApi=http://192.168.0.67:8080/admin/delete?username=carlos
```

---

[⬅ Voltar](../../../README.md)
