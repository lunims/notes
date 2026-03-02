# Burp Suite
* Paid-only:
	* Active web scanner
	* Fast Burp Intruder
	* The ability to load Burp extensions
	* https://portswigger.net/burp/pro/trial
* Proxy:
	* Intercepter -> intercept and modify HTTP requests before forwarding
* Intercepting responses
	* Proxy -> Proxy settings -> enable Intercept Responses under Response interception rules
	* Proxy -> Proxy settings -> Response modification rules -> Unhide hidden form fields
* Automatic request modification:
	* Burp Match and Replace: Proxy -> Proxy settings -> HTTP match and replace rules
* Repeating requests (Repeater):
	* Proxy -> History
	* burp offers ability to show original and modified request
	* send to Repeater (CTRL + SHIFT + R)
* URL Encoding:
	* right click text -> Convert Selection -> URL -> URL-encode key characters 
	* or CTRL + U
	* also supports automatic URL encoding as we type
* Decoding:
	* Decoder tab
	* also Burp Inspector tab
* **Burp Intruder**:
	* web fuzzer
	* free version is throttled at a speed of 1 request per second
	* [CTRL+I] -> send to Intruder
		* Positions:
			* place where we put payload position pointer 
			* done via § characters
		* Payloads:
			* Payload position
			* Payload type
				* Simple List: wordlist
				* Runtime file: loads line-by-line to avoid excessive memory consumption
				* Character substitution:  Lets us specify a list of characters and their replacements, and Burp Intruder tries all potential permutations
			* Payload configuration:
				* load the wordlists or manually add something
			* Payload processing:
				* determines fuzzing rules over loaded wordlist
				* for example: add a rule that skips any lines starting with a dot
			* Payload encoding:
				* enable or disable URL encoding
		* Settings tab:
			* Number of retries on network failure
			* Grep - Match: enables us to flag specific requests depending on the responses
				* for example, only look out for 200 status codes
* **Burp Scanner**
	* Target scope:
		* start a scan on a specific request from history
		* start a new scan on a set of targets
		* start a scan on items in scope
		* Target Scope:
			* Target>Site map shows all directories/file Burp has detected in various requests
			* right click -> Add to scope
	* Crawler:
		* Crawl and Audit
			* run scanner after crawler
		* Crawl
			* discover websites/links
		* Application login: give credentials to use in any login forms
	* Passive Scanner (Crawl and Audit):
		* does not send any new requests
		* missing HTML tags or potential DOM-XSS
		* only suggestions
	* Active Scanner (Crawl and Audit):
		* actively sends requests and tests for vulnerabilities
		* crawl -> passive scan -> checks each of identified vulns -> performs a JS analysis to identify further vulns -> fuzzes various identified insertion points
	* Reporting:
		* Issue>Report issues for this host on selected target
* Extensions in BApp Store

# ZAP
* free and open-source
* Proxy
	* Set breakpoint button (green) to red -> intercepted HTTP requests will come in a break TAB
	* can also be used in HUD in web browser (pre-configured browser)
		* step -> forward request and stay in intercepting mode
		* continue -> send all remaining requests
* Intercepting Responses:
	* click on step -> change response or CTRL + B
	* ZAP HUD lightbulb -> show all hidden input fields
* Automatic request modification:
	* ZAP Replacer: CTRL + R or Options menu
	* Initiators: enable us to select where Replacer rule options will be applied (default option: apply to all HTTP(S) messages)
* Repeating requests:
	* ZAP HUD also has history panel
	* only shows sent request
	* Open/Resend with Request Editor
	* In ZAP HUD: Replay in console to show in HUD windows or Replay in browser
* ZAP should automatically URL encode before sending the request
* Decoder: Encoder/Decode/Hash by CTRL + E 
	* custom tabs possible
* **ZAP Fuzzer**
	* Fuzz:
		* select request in history -> Attack>Fuzz
		* Locations:
			* select area to replace, then click Add
		* Payloads:
			* File: select wordlist
			* File Fuzzers: select wordlists from built-in databases of wordlists
			* Numberz: generates sequences of numbers with custom increment
			* also has built-in wordlists
		* Processors:
			* perform some processing (or encoding) on each word
			* like base64 etc.
			* Script type lets us select a custom script to run on every payload before attack
		* Options:
			* Concurrent Scanning Threads per Scan
			* Depth first: attempt all words from the wordlist on a single position before moving to the next
* ZAP Scanner:
	* Spider: 
		* Crawler, discover pages
		* Attack>Spider
	* Passive Scanner:
		* spider automatically runs passive scanner
		* identifies issues from source code
	* Active Scanner:
		* Click Active scan in ZAP HUD to run scan on all identified pages (in scope)
	* Reporting:
		* Report>Generate HTML report
* Extensions in ZAP Market place


### Proxy setup:
* Firefox preferences -> set up proxies (both listen to port 8080 on default)
* Also Foxy Proxy extensions helps when manually setting proxy
* Installing CA certificate:
	* Burp: http://burp
	* ZAP: Tools>Options>Network>Server Certificates
		* can also generate new one
	* can install in firefox under: about:preferences#privacy
		* Authorities tab -> click Import and select CA
* Proxying tools:
	* proxychains
		* linux tool
		* routes all traffic from any command-line tool to proxy we specify
### Usage proxychains:
* first edit /etc/proxychains.conf and comment out the final line
* add following line:
```shell-session
#socks4         127.0.0.1 9050
http 127.0.0.1 8080
```
*  also make use -q flag, which makes proxychains operate in "quiet mode"
```shell-session
lunims@htb[/htb]$ proxychains -q curl http://SERVER_IP:PORT

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Ping IP</title>
    <link rel="stylesheet" href="./style.css">
</head>
...SNIP...
</html>    
```
### Metasploit:
* example for robots.txt scanner
```shell-session
lunims@htb[/htb]$ msfconsole

msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(scanner/http/robots_txt) > set PROXIES HTTP:127.0.0.1:8080

PROXIES => HTTP:127.0.0.1:8080


msf6 auxiliary(scanner/http/robots_txt) > set RHOST SERVER_IP

RHOST => SERVER_IP


msf6 auxiliary(scanner/http/robots_txt) > set RPORT PORT

RPORT => PORT


msf6 auxiliary(scanner/http/robots_txt) > run

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
* 
# Burp Shortcuts:
* [CTRL+R]: Send to Repeater
* [CTRL+SHIFT+R]: Go to Repeater
* [CTRL+I]: Send to Intruder
* [CTRL+SHIFT+I]: Go to Intruder
* [CTRL+U]: URL Encode
* [CTRL+SHIFT+U]: URL Decode


# ZAP Shortcuts:
* [CTRL+B]: Toggle intercept on/off
* [CTRL+R]: Go to replacer
* [CTRL+E]: Go to encode/decode/hash