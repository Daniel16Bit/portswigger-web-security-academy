> 🌐 **English** | [Português](README.pt-BR.md)

# Server-Side Vulnerabilities

## Status

`COMPLETE`

## Module scope

Burp Suite "Apprentice"-level module. This learning path gives a simple introduction to common server-side vulnerabilities.

## Labs

| Lab | Difficulty | Status | Link |
|-----|-------------|--------|------|
| Path traversal | Apprentice | ✅ | [write-up](./01-PathTraversal/) |
| Access control | Apprentice | ✅ | [write-up](./02-AccessControl/) |
| Authentication | Apprentice | ✅ | [write-up](./03-Authentication/) |
| Server-side request forgery (SSRF) | Apprentice | ✅ | [write-up](./04-Server-SideRequestForgery(SSRF)/) |
| File upload vulnerabilities | Apprentice | ✅ | [write-up](./05-File-uploadVulnerabilities/) |
| OS command injection | Apprentice | ✅ | [write-up](./06-OS-commandInjection/) |
| SQL injection | Apprentice | ✅ | [write-up](./07-SQLInjection/) |

## General module notes

A pattern runs through every lab in this module: the server trusts something the client controls, or skips a check that should happen server-side.

- **Path traversal** — the server trusts a user-supplied file path and resolves `../` without validation.
- **Access control** — the server trusts a client-side role cookie (`admin=true`) or a URL/GUID parameter to decide who may access a resource.
- **Authentication** — the server creates the session before the second factor is validated, so the 2FA step isn't actually enforced.
- **SSRF** — the server trusts a client-supplied URL and makes the request on its behalf, reaching internal resources.
- **File upload** — the server trusts the client-supplied `Content-Type` (or does no validation at all) and executes the uploaded file.
- **OS command injection** — the server concatenates a user parameter directly into a shell command.
- **SQL injection** — the server concatenates user input directly into a SQL query.

The recurring root cause is a broken trust boundary: data the client controls is treated as if it were trustworthy, and validation/authorization that should be enforced server-side either happens on the client or doesn't happen at all. The corresponding defenses rhyme too — parameterized queries, allowlists, server-side authorization on every request, and never deriving trust from client-controlled values.

## References

- [PortSwigger Web Security Academy — Server-side topics](https://portswigger.net/web-security/all-topics)