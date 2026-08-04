> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# Lab: Remote code execution via web shell upload

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** File upload vulnerabilities //
**Status:** Solved //

## Goal

This lab has a vulnerable image upload feature that performs no validation of the uploaded files before storing them on the server's file system.

To solve the lab, it was necessary to upload a PHP web shell, execute it on the server to read the contents of `/home/carlos/secret`, and then submit the secret using the button provided by the lab.

The account provided by the exercise itself was used:

```
Username: wiener
Password: peter
```

## Recon

The lab description states that the upload feature does not validate the uploaded files, indicating a possible **File Upload** vulnerability capable of allowing remote code execution.

After uploading a test image and intercepting the request with Burp Suite, it was possible to analyze both the **POST** request (responsible for sending the file) and the **GET** request (later used to access the stored file).

This analysis showed that the uploaded files were served directly by the server, raising the hypothesis that it would be possible to upload a PHP file instead of an image and execute it remotely.

## Approach

- Logged in using the credentials provided by the lab.
- Accessed the avatar upload feature.
- Uploaded any image to understand how the application works.
- Intercepted the **POST** request using Burp Suite.
- Replaced the image content with a PHP file containing a simple web shell.
- Forwarded the modified request to the server.
- After the upload, intercepted the **GET** request responsible for accessing the uploaded image.
- Changed the file path to access the PHP file uploaded to the server.
- Upon accessing the file, the PHP code was executed by the server and returned the contents of `/home/carlos/secret`.
- The obtained secret was submitted to the lab, completing the challenge.

## Payload / Technique used

### Content sent via POST

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

### Original request

```http
GET /files/avatars/cat.png HTTP/1.1
```

### Modified request

```http
GET /files/avatars/exploit.php HTTP/1.1
```

## Evidence

![Evidence-01](imgs/Lab-01A.png)

![Evidence-02](imgs/Lab-01B.png)

## Result

The exploitation confirmed an **Unrestricted File Upload** vulnerability, allowing a PHP file to be uploaded and executed directly on the server.

As a consequence, it was possible to achieve remote code execution (RCE) and access sensitive information stored on the file system — in this case the contents of `/home/carlos/secret`, enough to complete the lab.

## Technical notes

### Why does the flaw occur?

- The application accepts user-uploaded files without validating their extension or content.
- The server stores these files in a web-accessible directory.
- Since the server interprets PHP files, an attacker can upload malicious code and execute it simply by accessing its URL.
- This behavior characterizes an **Unrestricted File Upload** vulnerability, frequently resulting in **Remote Code Execution (RCE)**.

### How to mitigate?

- Allow only specific file types (*allowlist*), instead of blocking known extensions.
- Validate the MIME type and the signature (*magic bytes*) of the uploaded file.
- Store uploads outside the application's public root.
- Configure the server to prevent script execution in upload directories.
- Rename uploaded files to avoid extension and path manipulation.
- Implement authentication and authorization mechanisms for access to uploaded files.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/file-upload) (link to the topic, not to the specific lab solution)
- OWASP Web Security Testing Guide - File Upload Testing
- CWE-434 - Unrestricted Upload of File with Dangerous Type
