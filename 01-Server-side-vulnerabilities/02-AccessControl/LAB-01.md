> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# Lab: Unprotected admin functionality

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Access control //
**Status:** Solved //

## Goal

The lab asks to access the admin panel and delete the user CARLOS.

## Recon

As stated in the lab description, this lab has an unprotected admin panel.
With that in mind, all we had to do was find the exposed URL.
The preceding topic explanation mentioned that there might also be internal files granting access hints (in this case, `robots.txt`), so basic logic was all that was needed.

## Approach

- The first step was a visual recon of the site to understand how it works.
- Based on what the lab description asked for and the hints it provided, it was possible to identify the next step.
- This time, the site alone was enough to carry out the approach.
- Aware of the path mentioned in the explanation (`/robots.txt`), I opened it and discovered the panel's URL.
- After reaching the panel, I simply deleted the requested user.

## Payload / Technique used

```
Simple URL-testing technique (/robots.txt), where we test URLs to check whether anything is exposed (/administrator-panel).
In this case there was, and it was obvious — but in other situations there is protection.
```

First, you need to understand what the `robots.txt` file is: extremely common and essential. It acts as a guide at the root of a site that tells search-engine crawlers (such as Googlebot) which pages or folders they **should not access**, helping optimize the site's crawl budget.
When a developer wants to hide an admin page from search engines, they do something like:

```
User-agent: *
Disallow: /administrator-panel
```

The intent is to say: "Google, don't show this in the search results."
What the developer doesn't realize is that they just revealed to any attacker exactly where the admin panel is. The file is public, so anyone (including an attacker / YOU) can read that info.

## Evidence

![Result](imgs/lab01.png)

## Result

In the end, I was able to delete the user CARLOS due to the security flaw present.

## Technical notes

Why doesn't this work on well-protected sites?
If the site has:

- Authentication on the panel (login + password)
- A firewall blocking unauthorized IPs
- Role/permission-based access control

...then even if you discover the URL through `robots.txt`, you won't be able to do anything, because there's a barrier. Knowing the URL alone is not enough — you still have to get through authentication.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/access-control) (link to the topic, not to the specific lab solution)
