---
layout: post
title: What Is SQL Injection - Overview
---

## What Is It?
SQL injection (SQLi) is a method of code injection that allows an attacker to run arbitrary SQL queries on a database. This allows the attacker to view data that they normally would not have access to, add, modify, or delete data, bypass web application authentication, or even access and run commands on the underlying Operating System. There are several different types of SQL injection such as error-based, time-based, union, and Boolean-based. 

Error-based SQLi refers to the application displaying error messages that an attacker could use to verify the vulnerability is exploitable as well as see the SQL statement they are injecting into. This error could look like this: *Invalid query: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '151' at line 1.*

Union-based SQLi refers to the usage of the SQL Union operator to combine the results of two or more Select statements into a single result. In practice, this means utilizing the union operator to add another select statement in addition to one the application is already performing. This second statement, however, must select the same number of fields as the first one.

Both error-based and Union-based SQLi rely on the application displaying the results somewhere on the page, however this will not always be the case. In situations where the application does not display the results of a query, an attacker can use blind-SQLi. Both time-based and Boolean-based are examples of blind SQLi.

Time-based SQLi refers to the usage of a delay on the application. If an attacker can consistently force the application to wait an arbitrary amount of time with the use of SQL operators such as waitfor in MSSQL or sleep in MYSQL, time-based SQLi has occurred.

Boolean-based SQLi refers to the usage of true/false statements to infer information. Although, this is a slow attack, it is still dangerous as it can be used to enumerate data.
The difference between them, for the most part, is self-explanatory. A search function that displays the input entered back onto the page is a good example of reflected XSS. 

## How is SQL Injection performed?
An attacker can begin by inserting a single or double quotation mark anywhere that a SQL query can be reasonably surmised to be taking place. This could be a search bar, login form, or directly inside a URL such as www.example.com/products.php?category=Gifts. The reason for this is to attempt to cause an error in the application.

Consider the following query: 

```sql
SELECT * FROM products WHERE category = ‘Gifts’ AND released = 1
```
Using the URL example from earlier (www.example.com/products.php?category=Gifts), adding a single quote after “Gifts” in the URL changes the query to this:

```sql
SELECT * FROM products WHERE category = ‘Gifts’’ AND released = 1
```
An error occurs due to this extra quotation mark. If the attacker then further changes the URL to *?category=Gifts’--*, the query changes to the following:
```sql
SELECT * FROM products WHERE category = ‘Gifts’--’ AND released = 1
```
This query will find all products where the category is gifts, ignoring the "AND" statement due to the usage of --. In this example, this will cause all products, released or not, in the gifts category to be displayed.	

## How is SQL Injection Prevented?
There are a few ways to remediate SQL injection. These include input sanitization, input validation, and prepared statements.

Input sanitization involves “cleaning up” user-supplied input that will be used in a SQL query. Further using the URL example from earlier, input sanitization would remove the quotation marks, double hyphens, or even any other characters entirely or escape them using the backslash character.

Input validation involves ensuring the data received is what is expected. If the application only needs to accept the “gifts” keyword for the category, this keyword could be checked for prior to executing a SQL query.

Prepared statements involve separating the code from the data. The query with a placeholder is first sent to the database. Following this, the data itself is sent to the database. Prepared statements allow the database to know the exact query to perform, preventing any user-supplied data from adding to, or modifying the query.
