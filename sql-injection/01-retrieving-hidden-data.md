# SQL Injection — Retrieving Hidden Data

**Category:** SQL Injection  
**Difficulty:** Apprentice  
**Link:** https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data  
**Date:** 2024-01-15  
**Status:** ✅ Solved

---

## What this lab tests

The application filters products by category using a URL parameter that gets concatenated directly into the SQL query. The goal is to modify the query to return all products, including ones flagged as unreleased in the database.

---

## How I solved it

**1.** The URL pattern was `/filter?category=Gifts`. Added a single quote `'` to the value — the app returned a 500 error, confirming the input goes straight into SQL without sanitization.

**2.** The original query is probably something like:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

**3.** Injected `' OR 1=1--` to make the WHERE clause always true and comment out the rest of the query, including the `released = 1` filter.

---

## Payload

```
/filter?category=Gifts' OR 1=1--
```

---

## What tried to block me

Nothing — introductory lab. Input goes straight into the query with no sanitization.

---

## How this maps to AWS

In an application backed by RDS, this type of injection has wider impact than it might seem:

- If the database user has excessive privileges, `UNION SELECT` can extract data from other tables — including tables with credentials or tokens stored by the application.
- In MySQL on RDS, `LOAD_FILE()` can read system files if `secure_file_priv` is not configured, potentially exposing environment variables that hold AWS credentials.
- Applications running on EC2 that pass credentials via environment variables (common in legacy setups) are especially at risk — SQLi leading to RCE exposes those variables directly.

The correct fix is always parameterized queries. WAF SQLi rules are a secondary layer, never the primary defense.

---

## What I learned

The `--` comment in MySQL needs a space after it in some contexts (`-- `). The `#` character also works as a comment in MySQL and is simpler to use in URLs. Worth testing both when one doesn't work.
