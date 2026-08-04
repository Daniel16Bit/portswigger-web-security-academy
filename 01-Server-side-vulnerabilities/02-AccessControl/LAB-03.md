> 🌐 **English** | [Português](LAB-03.pt-BR.md)

# Lab: User role controlled by request parameter

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Access control //
**Status:** Solved //

## Goal

- This lab has an admin panel at `/admin` that identifies administrators through a cookie that can be forged.
- Solve the lab by accessing the admin panel and using it to delete the user carlos.
- You can log in to your own account with the following credentials: `wiener:peter`.

## Recon

As stated in the lab description, this lab has an admin panel, but access is granted only to the correct role. With that in mind, our goal is to bypass that protection.

## Approach

- A visual recon of the application was performed to understand its structure and behavior.
- Based on the information provided by the lab description, it was possible to define the next step of the analysis.
- In this case there are two ways to proceed: the first is to intercept the request via Burp Suite, but we'll go with the second option.
- As the second approach, we simply logged in with the credentials provided in the lab description.
- After that, we analyzed the **Application** tab, looking for vulnerabilities in the **Storage** sub-tab.
- After analyzing its contents, we found that the Admin ROLE can be modified through cookies.
- With that, we set `admin=true`.

- Inspection of the browser's stored data (Application → Storage → Cookies).
- Tampering with a client-side cookie to escalate privileges (vertical privilege escalation).

In this lab it was not necessary to intercept requests with Burp (though the same result is achievable by editing the `Cookie` header in Proxy/Repeater). The exploitation consisted of editing the `Admin` cookie value in the browser's storage from `false` to `true`. Because the server trusts this client-controlled cookie as the source of truth for the user's role — instead of deriving the role server-side from the session — resending any request with `Admin=true` is enough to be treated as an administrator and reach `/admin`.

## Evidence

![Result](imgs/lab03.png)

## Result

User deleted successfully after breaking into the admin panel.

## Technical notes

- Never store roles/permissions in cookies. Use only a random session ID.
- Validate authorization on the server for every request to sensitive pages, querying the database or the session.
- Use server-side sessions (native PHP, Redis, database) instead of editable cookies.
- Sign/encrypt cookies if you need to store data on the client (e.g., a signed JWT).
- Test whether a regular user can reach `/admin` or manually modify cookies — that alone would have caught the flaw.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/access-control) (link to the topic, not to the specific lab solution)
