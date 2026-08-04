> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# Lab: Username enumeration via different responses

**Module:** Server-side vulnerabilities //
**Difficulty:** Apprentice //
**Category:** Authentication //
**Status:** Solved //

## Goal

This lab is vulnerable to username enumeration and password brute-force attacks.
It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, and then access their account page.

## Recon

Before anything else, we need to decide how we'll drive the attack.
To do so, we can use either of the two tools below:

- Burp Suite with the **Intruder** function — Pro is recommended, since Community throttles Intruder and is VERY slow.

(If you don't have Pro)

- OWASP ZAP (or just ZAP) with the **Fuzz** function. It's faster and free.

In this case, we'll use ZAP.

## Approach

- Located the `/login` URL and made a dummy login attempt (any username and password).
- With that, we captured the POST request and could run the attack.
- The attack is done in two stages: first enumerate a valid username, then brute-force that user's password.
- Once the password was cracked, it was possible to log in to the requested account.

## Payload / Technique used

### 1. Username enumeration

A fuzzing attack was run against the `username` parameter only, keeping the `password` fixed. The valid username is identified by the difference in the server's response (a different error message for existing vs. non-existing accounts).

```
POST /login HTTP/1.1

username=§USER§
password=anything
```

### 2. Password brute-force

With the valid username in hand, a second attack was run — this time fixing `username` and fuzzing `password` against the candidate passwords wordlist. The valid password is the one that breaks the pattern (e.g., a `302` redirect or a response without the error message).

```
POST /login HTTP/1.1

username=<valid_user>
password=§PASSWORD§
```

Both stages use the wordlists provided by the lab.

## Evidence

![LabComplete](imgs/Lab01-A.png)
----------------------------
![LabComplete](imgs/Lab01-B.png)

## Result

The attack made it possible to identify a valid username through the differences in the messages returned by the server, and then brute-force the corresponding password to access the account.

## Technical notes

Why does the flaw occur?

- The vulnerability occurs because the application returns different responses for existing and non-existing users during authentication.
- These differences can appear in the displayed messages, the HTTP status code, the response length, or even the processing time.
- This behavior lets an attacker discover which accounts are valid before even trying to crack their passwords, significantly reducing the effort needed for a brute-force attack.

How to mitigate?

- Use generic error messages, such as "Invalid username or password", regardless of the cause of the failure.
- Keep responses with similar length and processing time for all authentication attempts.
- Implement rate limiting.
- Temporarily lock accounts after several consecutive failed attempts.
- Use multi-factor authentication (MFA) to reduce the impact of a leaked password.
- Monitor repeated login attempts to detect possible automated attacks.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/authentication) (link to the topic, not to the specific lab solution)
