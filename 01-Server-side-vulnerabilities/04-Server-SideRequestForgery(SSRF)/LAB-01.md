> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# LAB: Basic SSRF against the local server

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Server-side request forgery (SSRF) //
**Status:** Solved //

## Goal

This lab has a stock check feature that fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user Carlos.

## Recon

The lab provides a stock check feature (Check stock) that makes a request to an internal server responsible for querying product availability.

By intercepting the request with Burp Suite, it was possible to identify the `stockApi` parameter, which tells the server which address to query for the stock information.

This characteristic indicates a possible Server-Side Request Forgery (SSRF) vulnerability, since the application makes HTTP requests on the user's behalf using the URL provided in the parameter.

## Approach

- Accessed any product page.
- Selected the Check stock option to generate the request to the server.
- Intercepted the `POST /product/stock` request using Burp Suite.
- Identified the `stockApi` parameter, which pointed to the internal stock server.
- Changed its value to `http://localhost/admin`.
- The response returned the application's internal admin panel, revealing the endpoint responsible for deleting users.
- Identified the URL `/delete?username=carlos`.
- Modified the `stockApi` parameter again to point directly to that endpoint.
- After sending the request, the user carlos was removed and the lab was solved.

## Payload / Technique used

```
POST /product/stock HTTP/1.1

stockApi=http://localhost/admin
```

After identifying the admin endpoint:

```
POST /product/stock HTTP/1.1

stockApi=http://localhost/admin/delete?username=carlos
```

## Evidence

![Evidence-01](imgs/Lab01%20-%20A.png)
![Evidence-02](imgs/Lab01%20-%20B.png)

## Result

The vulnerability allowed the server to make HTTP requests to internal resources that are not accessible externally.

## Technical notes

### Why does the flaw occur?

- The application accepts a user-supplied URL and makes the request directly on the server side, without properly validating the destination.
- Since the server has access to the internal network, an attacker can use it to reach resources that would normally not be available externally.
- This allows querying internal services, accessing admin interfaces and, in some cases, interacting with other systems in the infrastructure.

### How to mitigate?

- Never let the client directly control the destination URL of the requests.
- Use an allowlist containing only the necessary domains.
- Block requests aimed at internal addresses, such as `localhost`, `127.0.0.1` and private networks.
- Strictly validate the protocol and destination of the received URLs.
- Segment the internal network and restrict services' access to administrative resources.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/ssrf) (link to the topic, not to the specific lab solution)
