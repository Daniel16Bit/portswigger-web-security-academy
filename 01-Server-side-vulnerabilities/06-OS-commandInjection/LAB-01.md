> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# Lab: OS command injection, simple case

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** OS command injection //
**Status:** Solved //

## Goal

This lab contains an **OS Command Injection** vulnerability in the product stock check feature.

To solve the lab, it was necessary to exploit this vulnerability to run the `whoami` command on the server and identify the user running the application.

## Recon

The lab description indicates the presence of an **OS Command Injection** vulnerability.

Based on the guidance provided by PortSwigger itself during the lab, the requests responsible for the product stock check were analyzed.

By using the **Check stock** feature and intercepting the communication with Burp Suite, the following request was identified:

```http
POST /product/stock
```

This request sent the parameters:

```text
productId=3&storeId=3
```

Since these parameters were used by the server to query the product stock, the hypothesis arose that one of them might be concatenated directly into an operating-system command without proper validation.

## Approach

- Accessed any product page.
- Selected the **Check stock** option.
- Intercepted the `POST /product/stock` request using Burp Suite.
- Forwarded the request to Repeater.
- Modified the `storeId` parameter, appending a shell operator (`|`) followed by the `whoami` command.

The payload used was:

```text
productId=3&storeId=3|whoami
```

- After resending the request, the server executed the additional command.
- The response returned the user running the application, confirming the **OS Command Injection** vulnerability.
- With that, the lab was solved.

## Payload / Technique used

### Original request

```text
productId=3&storeId=3
```

### Payload used

```text
productId=3&storeId=3|whoami
```

## Evidence

![Evidence-01](imgs/Lab-01A.png)

![Evidence-02](imgs/Lab-01B.png)

## Result

The exploitation confirmed an **OS Command Injection** vulnerability, allowing arbitrary commands to be executed on the operating system through a user-controlled parameter.

It was possible to run the `whoami` command, obtaining the user running the application on the server and proving remote command execution.

## Technical notes

### Why does the flaw occur?

The application uses a user-supplied parameter to build an operating-system command without performing proper validation or sanitization.

The `|` operator is interpreted by the shell as a command-chaining (pipe) operator, causing the original command to run normally and then also execute the command supplied by the attacker.

This practice lets malicious users run arbitrary commands on the operating system, potentially resulting in a full server compromise.

### How to mitigate?

- Avoid running operating-system commands whenever there is an alternative using the language's native functions.
- Never concatenate user-controlled data directly into system commands.
- Strictly validate all received parameters.
- Use allowlists for expected values.
- When it's necessary to run external commands, use APIs that don't invoke the system shell.
- Run the application under the principle of least privilege, limiting the impact of a possible exploitation.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/os-command-injection) (link to the topic, not to the specific lab solution)
- OWASP Web Security Testing Guide - Command Injection
- CWE-78 - Improper Neutralization of Special Elements used in an OS Command
