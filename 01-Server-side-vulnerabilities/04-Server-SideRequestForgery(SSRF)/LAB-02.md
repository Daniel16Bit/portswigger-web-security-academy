> 🌐 **English** | [Português](LAB-02.pt-BR.md)

# LAB: Basic SSRF against another back-end system

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Server-side request forgery (SSRF) //
**Status:** Solved //

## Goal

This lab has a stock check feature that fetches data from an internal system.

To solve the lab, use the stock check feature to scan the internal `192.168.0.X` range for an admin interface on port 8080, then use it to delete the user carlos.

## Recon

Before exploiting the vulnerability, it was necessary to analyze how the stock check feature communicated with the internal server.

When selecting the **Check stock** option, the application sent a `POST /product/stock` request.

Intercepting that request, it was possible to identify the `stockApi` parameter, responsible for defining which address the server would query.

Since the lab's goal was to locate an admin server inside the internal `192.168.0.X` network, a fuzzing technique was used to discover which IP address hosted the admin interface.

## Approach

- Accessed any product page.
- Clicked **Check stock** to generate the request.
- Intercepted the `POST /product/stock` request.
- Sent the request to **OWASP ZAP** and used the **Fuzz** tool to automate the attempts.
- Set the last octet of the IP address (`192.168.0.X`) as the variable parameter.
- Ran the fuzzing, testing addresses from `192.168.0.1` to `192.168.0.255`.
- While analyzing the responses, identified that the address `192.168.0.13` returned the admin interface.
- Then changed the `stockApi` parameter to reach the user-deletion endpoint:
  `http://192.168.0.13:8080/admin/delete?username=carlos`
- Sent the request and the user **carlos** was removed, solving the lab.

## Payload / Technique used

First request used to identify the admin server:

```http
POST /product/stock HTTP/1.1

stockApi=http://192.168.0.1:8080/admin
```

Fuzzing was performed over the last octet of the IP address:

```
192.168.0.1
192.168.0.2
192.168.0.3
...
192.168.0.13
...
192.168.0.255
```

After identifying the correct server:

```http
POST /product/stock HTTP/1.1

stockApi=http://192.168.0.13:8080/admin/delete?username=carlos
```

---

## Evidence

![Evidence-01](imgs/Lab02%20-%20A.png)
![Evidence-02](imgs/Lab02%20-%20B.png)

## Result

The vulnerability made it possible to use the application's server to reach different machines on the internal network.

## Technical notes

### Why does the flaw occur?

- The application lets the client control the address to which the server will make an HTTP request.
- Since the server has access to the internal network, an attacker can use it to perform scans (*port scanning* and *host discovery*) on equipment that is not accessible externally.
- After locating a vulnerable internal service, it's possible to interact with it directly through the application, characterizing a **Server-Side Request Forgery (SSRF)** vulnerability.

### How to mitigate?

- Never let users directly control the destination of the requests made by the server.
- Use allowlists containing only the authorized domains and addresses.
- Block requests to private networks, such as `192.168.0.0/16`, `172.16.0.0/12` and `10.0.0.0/8`.
- Restrict the application's access to internal resources that are not strictly necessary.
- Monitor unusual requests that may indicate internal network enumeration attempts.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/ssrf) (link to the topic, not to the specific lab solution)
