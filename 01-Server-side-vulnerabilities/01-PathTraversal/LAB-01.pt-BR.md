> 🌐 [English](LAB-01.md) | **Português**

# Lab: File path traversal, simple case

**Módulo:** Server-side vulnerabilities //
**Dificuldade:** Apprentice //
**Categoria:** Path traversal //
**Status:** Resolvida //

## Objetivo

Acessar o conteúdo do caminho `/etc/passwd`.

## Reconhecimento

Assim como informado pelo enunciado, este lab tem uma path traversal vulnerability na hora de mostrar as imagens dos produtos.

## Abordagem

- O primeiro passo foi efetuar um reconhecimento visual do site e de como ele funciona.
- Com base no que o enunciado pediu e nas dicas que ele nos dá, foi possível identificar o próximo passo.
- Usando o Burp Suite e o site ao mesmo tempo, usei o filtro do proxy para capturar as requisições que carregam as imagens do site.
- Ao capturar o `GET /image?filename=1.jpg`, foi então que a vulnerabilidade foi explorada.

## Payload / Técnica utilizada

```
A forma como executei foi modificar o GET da imagem para acessar o caminho pedido.
Enviei a requisição GET para o Repeater e, a partir disso, consegui acessar.

Payload usado: ../../../etc/passwd  [RESULTADO: GET /image?filename=../../../etc/passwd HTTP/2]
```

O site tem uma função que carrega imagens baseada em um parâmetro na URL. Internamente, o servidor pega esse parâmetro e concatena com um diretório fixo para montar o caminho completo do arquivo. Como o servidor não valida o que é passado, eu posso usar `../` para subir diretórios — saindo da pasta de imagens — e acessar qualquer arquivo do sistema, como `/etc/passwd`. O sistema operacional resolve os `../` como "voltar uma pasta", então `../../../etc/passwd` a partir da pasta de imagens acaba apontando para `/etc/passwd`.

## Evidência

![Resultado](imgs/resultado.png)

## Resultado

Ao final, consegui acessar um arquivo essencial para o sistema operacional, que armazena os detalhes das contas dos usuários.

## Observações técnicas

Por que o servidor não bloqueia? Porque o desenvolvedor não implementou nenhuma sanitização no parâmetro `filename`. Ele deveria ter feito algo como:

- Verificar se o caminho final realmente começa com `/var/www/html/images/`
- Remover ou bloquear `..` do parâmetro
- Usar uma lista branca (whitelist) de arquivos permitidos

Mas como não fez, o atacante (EU) consegue "escapar" do diretório restrito e navegar por todo o sistema de arquivos do servidor.

## Referências

- [PortSwigger Web Security Academy](https://portswigger.net/support/using-burp-to-test-for-path-traversal-vulnerabilities) (link para o tópico, não para a lab específica com solução)