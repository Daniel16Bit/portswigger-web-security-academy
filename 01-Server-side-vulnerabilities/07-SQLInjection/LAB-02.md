> 🌐 **English** | [Português](LAB-02.pt-BR.md)

# Lab: SQL injection vulnerability allowing login bypass

**Module:** SQL injection //
**Difficulty:** Apprentice //
**Category:** SQL Injection //
**Status:** Solved //

## Goal

This lab contains a **SQL Injection** vulnerability in the authentication feature.

To solve the lab, it was necessary to exploit this vulnerability to log in as the **administrator** user, without knowing their password.

## Recon

The lab provides a login page composed of the **Username** and **Password** fields.

As stated by the lab description, the application is vulnerable to **SQL Injection** during login, indicating that the values supplied by the user are likely inserted directly into the SQL query responsible for authentication.

This characteristic suggests the possibility of altering the query's structure to bypass the password check.

## Approach

- Accessed the application's login page.
- In the **Username** field, entered a SQL Injection payload to close the string and comment out the rest of the query.
- In the **Password** field, entered `--`, since its check would be commented out by the injection (any value would work here, as the password is never evaluated).
- After submitting the form, the application authenticated the session as the **administrator** user, completing the lab.

## Payload / Technique used

### Credentials used

**Username**

```text
administrator'--
```

**Password**

```text
--
```

### Query logic example

Query expected by the application:

```sql
SELECT *
FROM users
WHERE username = 'administrator'
AND password = 'password';
```

Query after the injection:

```sql
SELECT *
FROM users
WHERE username = 'administrator'--'
AND password = '--';
```

Since `--` starts a comment in SQL, everything after it — including the `AND password = '...'` check — is ignored by the database. The query effectively becomes `WHERE username = 'administrator'`, so authentication succeeds based on the username alone. The value entered in the password field (`--`) is never evaluated by the database, since it falls inside the commented-out portion.

## Evidence

![Evidence-01](imgs/Lab-02.png)

## Result

The exploitation confirmed a **SQL Injection** vulnerability in the authentication feature.

It was possible to completely bypass the password check and authenticate as the **administrator** user, demonstrating that the application concatenates user-controlled data directly into SQL queries.

## Technical notes

### Why does the flaw occur?

The application builds the SQL query using the values supplied by the user directly, without using parameterized queries.

By entering the payload:

```text
administrator'--
```

the string is closed prematurely and the `--` operator turns the rest of the query into a comment.

This way, the condition responsible for checking the password is no longer executed, allowing authentication based only on the username.

### How to mitigate?

- Use **Prepared Statements** (parameterized queries) for all database queries.
- Never concatenate user-supplied input directly into SQL statements.
- Validate and sanitize the received data.
- Implement authentication using parameterized queries and secure credential comparison.
- Apply the principle of least privilege to the accounts used by the application to access the database.
- Avoid detailed error messages that could aid in exploiting the vulnerability.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection) (link to the topic, not to the specific lab solution)
- OWASP Web Security Testing Guide - SQL Injection
- CWE-89 - Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')
