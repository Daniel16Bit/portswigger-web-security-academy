> 🌐 [English](README.md) | **Português**

# Server-Side Vulnerabilities

## Status

`COMPLETO`

## Escopo do módulo

Módulo de nível "APRENDIZ" (Apprentice) do Burp. Este learning path introduz de forma simples vulnerabilidades comuns no lado do servidor.

## Labs

| Lab | Dificuldade | Status | Link |
|-----|-------------|--------|------|
| Path traversal | Apprentice | ✅ | [resolução](./01-PathTraversal/) |
| Access control | Apprentice | ✅ | [resolução](./02-AccessControl/) |
| Authentication | Apprentice | ✅ | [resolução](./03-Authentication/) |
| Server-side request forgery (SSRF) | Apprentice | ✅ | [resolução](./04-Server-SideRequestForgery(SSRF)/) |
| File upload vulnerabilities | Apprentice | ✅ | [resolução](./05-File-uploadVulnerabilities/) |
| OS command injection | Apprentice | ✅ | [resolução](./06-OS-commandInjection/) |
| SQL injection | Apprentice | ✅ | [resolução](./07-SQLInjection/) |

## Anotações gerais do módulo

Um padrão atravessa todos os labs deste módulo: o servidor confia em algo que o cliente controla, ou pula uma verificação que deveria acontecer no lado do servidor.

- **Path traversal** — o servidor confia em um caminho de arquivo fornecido pelo usuário e resolve `../` sem validação.
- **Access control** — o servidor confia em um cookie de cargo controlado pelo cliente (`admin=true`) ou em um parâmetro de URL/GUID para decidir quem pode acessar um recurso.
- **Authentication** — o servidor cria a sessão antes de validar o segundo fator, então a etapa de 2FA não é de fato imposta.
- **SSRF** — o servidor confia em uma URL fornecida pelo cliente e realiza a requisição em seu nome, alcançando recursos internos.
- **File upload** — o servidor confia no `Content-Type` informado pelo cliente (ou não valida nada) e executa o arquivo enviado.
- **OS command injection** — o servidor concatena um parâmetro do usuário diretamente em um comando de shell.
- **SQL injection** — o servidor concatena a entrada do usuário diretamente em uma consulta SQL.

A causa raiz recorrente é uma fronteira de confiança quebrada: dados que o cliente controla são tratados como confiáveis, e a validação/autorização que deveria ser imposta no servidor ou acontece no cliente ou simplesmente não acontece. As defesas correspondentes também rimam — consultas parametrizadas, allowlists, autorização server-side a cada requisição, e nunca derivar confiança de valores controlados pelo cliente.

## Referências

- [PortSwigger Web Security Academy — Tópicos server-side](https://portswigger.net/web-security/all-topics)
