> 🌐 **English** | [Português](LAB-04.pt-BR.md)

# Lab: User ID controlled by request parameter, with unpredictable user IDs

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Access control //
**Status:** Solved //

## Goal

This lab has a horizontal privilege escalation vulnerability on the user account page, but it identifies users through GUIDs.

To solve the lab, find the GUID of the user "carlos" and then submit his API key as the solution.

You can log in to your own account with the following credentials: `wiener:peter`.

## Recon

As mentioned earlier, this lab contains an escalation vulnerability (**Broken Access Control**) via a GUID flaw. With that in mind, we'll inspect URLs and files using element inspection.

## Approach

- A visual recon of the application was performed to understand its structure and behavior.
- Based on the information provided by the lab description, it was possible to define the next step of the analysis.
- First, element inspection was used to check whether there was anything unusual that could be leveraged.
- Since nothing was found, we moved on to examining the site in more detail.
- On closer inspection, a flaw was found in the URLs when accessing blog posts: each author's name links to `/blogs?userId=<GUID>`, leaking the user's GUID in public content.
- Having carlos's GUID, I requested `/my-account?id=<carlos_GUID>`. The server returned his account page — including his API key — without checking whether the GUID belonged to my session.
- I extracted the API key and submitted it as the lab solution.

## Payload / Technique used

- Web application recon (passive GUID enumeration from public blog content).
- IDOR on `/my-account?id=<GUID>` — direct access to another user's account page.

No automated tooling or request tampering beyond swapping the `id` parameter was required. The GUID (unpredictable by design) was leaked in the application's own HTML, defeating the "unpredictable ID" mitigation.

## Evidence

![Result](imgs/lab04.png)

## Result

carlos's GUID was recovered from the public blog links, used to access his account page via IDOR, and his API key was extracted and submitted as the lab solution.

## Technical notes

Horizontal Broken Access Control (IDOR). The `/my-account` route does not implement:

- Ownership verification of the resource — the server doesn't compare the `userId` of the authenticated session with the `userId` passed in the `id` parameter.
- Authorization middleware — there's no layer intercepting the request to validate whether the user may access the requested profile.
- Session-to-resource binding — the route accepts any valid GUID in the `id` parameter, regardless of who is logged in.
- Identifier obfuscation in public content — user GUIDs are exposed in the application's HTML elements (comments, post links, metadata), allowing passive enumeration.

The server accepts GET requests to `/my-account?id=ANY_GUID` from any authenticated user, without verifying whether the requested GUID belongs to the active session.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/access-control) (link to the topic, not to the specific lab solution)
