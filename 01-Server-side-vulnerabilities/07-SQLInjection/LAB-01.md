> 🌐 **English** | [Português](LAB-01.pt-BR.md)

# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Module:** SQL injection //
**Difficulty:** Apprentice //
**Category:** SQL Injection //
**Status:** Solved //

## Goal

This lab contains a **SQL Injection** vulnerability in the product category filter.

Whenever a category is selected, the application runs a query similar to the following:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

To solve the lab, it was necessary to modify this query so that products not yet published (*unreleased products*) were also displayed.

## Recon

When selecting a product category, it was observed that the URL contained the parameter responsible for the filter:

```text
/filter?category=Gifts
```

The lab description states that this parameter is used directly in the query's **WHERE** clause.

This indicates that, if the user input is not properly validated, it's possible to alter the query's logic through a SQL injection.

## Approach

- Selected any product category.
- Identified the `category` parameter in the URL.
- Modified its value by appending a SQL Injection payload.
- The payload used closes the original string, adds an always-true condition (`OR 1=1`), and comments out the rest of the query using `--`.
- After sending the request, the application started returning all registered products, including those not yet published.
- Displaying the hidden products completed the lab.

## Payload / Technique used

### Original URL

```text
/filter?category=Gifts
```

### Payload used

```text
/filter?category=Gifts'+OR+1=1--
```

### Original query

```sql
SELECT *
FROM products
WHERE category = 'Gifts'
AND released = 1;
```

### Query after the injection

```sql
SELECT *
FROM products
WHERE category = 'Gifts'
OR 1=1--'
AND released = 1;
```

## Evidence

![Evidence-01](imgs/Lab-01.png)

## Result

The exploitation confirmed a **SQL Injection** vulnerability, allowing the logic of the query's **WHERE** clause to be altered.

By using an always-true condition (`OR 1=1`), it was possible to bypass the restriction responsible for showing only published products, making the application return the hidden products (*unreleased products*) as well, completing the lab.

## Technical notes

### Why does the flaw occur?

The application concatenates the user-supplied value directly into the SQL query, without validating it or using parameterized queries.

The payload used closes the category string (`'`), adds the `OR 1=1` condition (which is always true), and uses `--` to comment out the rest of the SQL statement.

As a result, the clause responsible for filtering only published products (`AND released = 1`) is no longer considered by the database.

### How to mitigate?

- Use **parameterized queries (Prepared Statements)** in all database interactions.
- Never concatenate user-controlled input directly into SQL queries.
- Validate the received parameters using allowlists whenever possible.
- Apply the principle of least privilege to the accounts used by the application to access the database.
- Implement proper error handling to avoid exposing information about the query structure.

## References

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection) (link to the topic, not to the specific lab solution)
- OWASP Web Security Testing Guide - SQL Injection
- CWE-89 - Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')
