# Web Fuzzing
* Fuzzing vs. Brute-Forcing:
	* Fuzzing casts a wider net. It involves feeding unexpected inputs with malformed data etc. The goal is to see how the application reacts to these strange inputs and uncover potential vulnerabilities
	* Brute-forcing is a more targeted approach. It focuses on systemically trying many possibilities for a specific value, such as a password or an ID number
* Why Fuzz Web Apps?
	* Uncover Hidden Vulnerabilities
	* Automating Security Testing
	* Simulating Real-world Attacks
	* Strengthening Input Validation
	* Improving Code Quality
	* Continious Security
* Tooling:
	* Python (+pip) and Go should be installed
* **ffuf**
	* preinstalled on kali
	* Use cases:  
		* Directory and file enum
		* Parameter Discovery
		* Brute-Force attack
```
lunims@htb[/htb]$ go install github.com/ffuf/ffuf/v2@latest
```
* **gobuster**
	* preinstalled on kali
	* Use cases:
		* Content Discovery
		* DNS Subdomain Enumeration
		* WordPress Content Detection
```
lunims@htb[/htb]$ go install github.com/OJ/gobuster/v3@latest
```
* **FeroxBuster**
	* Use cases:
		* Recursive Scanning
		* Unlinked Content Discovery
		* High-Performance Scans
```
lunims@htb[/htb]$ curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | sudo bash -s $HOME/.local/bin
```
* **wfuzz/wenum**
	* Use cases:
		* Directory and File Enumeration
		* Parameter Discovery
		* Brute-Force Attack
```
lunims@htb[/htb]$ pipx install git+https://github.com/WebFuzzForge/wenum lunims@htb[/htb]$ pipx runpip wenum install setuptools
```

# Directory and File Fuzzing
* wordlists (most common of seclists)
	* Discovery/Web-Content/common.txt
	* Discovery/Web-Content/directory-list-2.3-medium.txt
	* Discovery/Web-Content/raft-large-directories.txt
	* Discovery/Web-Content/big.txt (both directories and files)
* ffuf Usage:
	1. Wordlist
	2. URL with FUZZ keyword
	3. Rquests
	4. Response Analysis
* Directory Fuzzing
	* -w -> wordlist flag
	* -u -> url flag
```
lunims@htb[/htb]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP:PORT/FUZZ
```
* File Fuzzing:
	* -e -> extension flag that gets appended
```
lunims@htb[/htb]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://IP:PORT/w2ksvrus/FUZZ -e .php,.html,.txt,.bak,.js -v
```
### Recursive Fuzzing
* how does it work?
	1. Initial Fuzzing: start at top (root `/`), fuzzer analyzes server responses looking for successful requests
	2. Directory Discovery and Expansion: when valid directory is found, it creates a new branch for that directory, essentially appending the directory name to the base URL
	3. Iterative Depth: process repeats itself for each discovered directory
* Recursive Fuzzing with ffuf 
	* -recursion flag
	* Be responsible!
		* -recursion-depth: allows you to specify the maximum depth for recursive exploration
		* -rate: can control the rate at which ffuf sends requests per second
		* -timeout: sets the timeout for individual requests, helping to prevent the fuzzer from hanging on unresponsive targets

# Parameter and Value Fuzzing
* GET parameters via ? in URL
	* often queries like search or filtering
* POST Parameters in request body
* **wenum**
```
lunims@htb[/htb]$ pipx install git+https://github.com/WebFuzzForge/wenum lunims@htb[/htb]$ pipx runpip wenum install setuptools
```
Usage:
```
wenum -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404 -u "http://IP:PORT/get.php?x=FUZZ"
```
* --hc 404 -> hides responses with 404, since wenum will log every request it makes by default
* for ffuf this is -fc flag
* The target value is specified with the FUZZ keyword just like with ffuf
POST Requests:
```
ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v
```
* -d flag which tells ffuf that the payload y=FUZZ should be sent

# Virtual Host and Subdomain Fuzzing
* basically can look at notes for other module
* can use gobuster to fuzz for Vhosts
* gobuster also has dns functionality for finding subdomains
```
gobuster dns --domain inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

# Filtering Fuzzing Output
* gobuster
	* -s (include) -> include only responses with the specified status codes
	* -b (exclude) -> exclude responses with the specified status codes
	* --exclude-lenght -> exclude responses with specific content length
* ffuf
	* -mc (match code) -> include only responses that match specified status codes
	* -fc (filter code) -> exclude responses that match specified status codes
	* -fs (filter size) -> exclude responses with a specific size
	* -ms (match size) -> include only responses that match specific size
	* -fw (filter out number of words in response) -> exclude responses with specified number of words in response
	* -mw (match words)
	* -fl (filter line)
	* -ml (match line)
	* -mt (match time) -> Include only responses that meet a specific time-to-first-byte (TTFB) condition
* wenum
	* --hc (hide code)
	* --sc (show code)
	* --hl (hide length)
	* --sl (show length)
	* --hw (hide word)
	* --sw (show word)
	* --hs (hide size)
	* --ss (show size)
	* --hr (hide regex)
	* --sr (show regex)
	* --filter / -hard-filter -> General-purpose filter to show/hide responses or prevent their post-processing using a regular expression

# Validating Findings
* Why Validate?
	* Confirming Vulnerabilities
	* Understanding Impact
	* Reproducing the Issue
	* Gather Evidence
* Manual Verification
	1. Reproducing the request: use a tool like curl or web browser to manually send the same request that triggered the unusual response
	2. Analyzing the Response: examine response to confirm whether it indicates vulnerability. Look for error messages, unexpected content or behavior that deviates from the expected norm
	3. Exploitation: if the finding seems promising, attempt to exploit the vulnerability in a controlled environment to assess its impact and severity. This step should be performed with caution and only after obtaining proper auhtorization
* to responsibly confirm vulnerabilities we could for example examine the headers using curl -I (see if there is actual Content in Content-lenght)
Skill Assesment:
First enumerate directories (i used ffuf)
```
ffuf -u http://154.57.164.72:32544/FUZZ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt
```
We found directory ui-hiddenmember
When opening in browser we see that this directory lists backup.tar.gz
Use curl to get its Headers:
```
curl -I http://154.57.164.72:32544/ur-hiddenmember/backup.tar.gz
```

# Web APIs
* a web API is (Web Application Programming Interface) is a set of rules that enable different software applications to communicate over the web
* essentially serves as a bridge between a server and a client
* Representational State Exchange Transfer (REST)
	* HTTP methods (GET, POST, PUT, DELETE)
	* CRUD -> Create, Read, Update, Delete
	* often exchange data in json or xml
* Simple Object Access Protocol (SOAP)
	* more formal and standardized 
	* uses xml to define messages which are encapsulated in SOAP envelopes and transmitted over network protocols like HTTP or SMTP
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/"> <soapenv:Header/> <soapenv:Body> <tem:GetStockPrice> <tem:StockName>AAPL</tem:StockName> </tem:GetStockPrice> </soapenv:Body> </soapenv:Envelope>
```
* GraphQL
	* single endpoint
	* eliminates problem of over- or under-fetching data
	* Example query:
```graphql
query 
	{ 
		user(id: 123) 
	{ 
		name email 
	} 
}
```
### Identifying Endpoints
* REST
	* endpoints are structured as URLs representing the resources you want to access or manipulate
		* /users
		* /users/123
		* /products
		* /products/456
	* Parameters are used to modify the behavior of API requests or provide additional information
		* Query Parameters -> `/users?limit=10&sort=name`
		* Path Parameters -> `/products/{id}pen_spark`
		* Request Body Parameters -> `{ "name": "New Product", "price": 99.99 }`
	* Discovering:
		* API Documentations: mostly OpenAPI or RAML
		* Network Traffic Analysis
		* Parameter Name Fuzzing
* SOAP
	* SOAP typically expose a single endpoint for a resource
	* parameters are defined within SOAP message (XML)
	* Example:
		* keywords(string): search term to use
		* author(string): name of the author (optional)
		* genre(string): genre of the book (optional)
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:lib="http://example.com/library"> 
	<soapenv:Header/> 
	<soapenv:Body> 
		<lib:SearchBooks> 
			<lib:keywords>cybersecurity</lib:keywords> 
			<lib:author>Dan Kaminsky</lib:author> 
		</lib:SearchBooks> 
	</soapenv:Body> 
</soapenv:Envelope>
```
* In this request keywords is set to cybersecurity for example.
* Discovering SOAP Endpoints
	* WSDL Analysis: WSDL file is most valuable resource for understanding a SOAP API. It describes available operations, input parameters, output parameters, data types for parameters and the location of the SOAP endpoint
	* Network Traffic Analysis
	* Fuzzing for Parameter Names and Values
* GraphQL
	* single endpoint (often called /graphql)
	* uses a unique query language to specify data requirements
	* GraphQL Queries:
		* Field: Represents a specific piece of data you want to retrieve (example: name, email)
		* Relationship: Indicates a connection between different types of data (e.g. a user's posts)
		* Nested Object: field that returns another object, allowing you to traverse deeper into the data graph (example: post {title, body})
		* Argument: Modifies the behavior of a query or field (e.g. filtering, sorting, pagination) -> posts(limit: 5) retrieves first 5 posts of a user
Example: 
```graphql
query 
	{ 
		user(id: 123) { 
			name 
			email 
			posts(limit: 5) { 
				title 
				body 
			} 
		} 
	}
```
* GraphQL Mutations:
	* mutations are counterpart to queries designed to modify data
	* arguments:
		* Operation: action to perform (e.g. createPost, updateUser, deleteComment)
		* Argument: Input data required for the operation (e.g. title and body for a new post)
		* Selection: Fields you want to retrieve in the response after the mutation completes
```graphql
mutation { createPost(title: "New Post", body: "This is the content of the new post") { id title } }
```
* Discovering Queries and Mutations:
	* Introspection: GraphQL's introspection system is a powerful tool for discovery, by sending introspection request to GraphQL endpoint, one can retrieve a complete schema describing the API's capabilities
	* API Documentation: Tools like GraphiQL or GraphQL Playground can help
	* Network Traffic Analysis
### API Fuzzing
* Types of API Fuzzing:
	* Parameter Fuzzing -> focuses on testing different values for API parameters
	* Data Format Fuzzing -> specifically targets data formats by manipulating the structure, content or encoding of the data 
	* Sequence Fuzzing -> examines how an API responds to sequences of requests (manipulate order, timing or parameters of API calls)
* Fuzzing APIs with apifuzzer
```shellsession
lunims@htb[/htb]$ git clone https://github.com/PandaSt0rm/webfuzz_api.git lunims@htb[/htb]$ cd webfuzz_api lunims@htb[/htb]$ pip3 install -r requirements.txt
```
Usage:
```shellsession
lunims@htb[/htb]$ python3 api_fuzzer.py http://IP:PORT
```

# Skill Assessment
* All Fuzzing can be done with seclists/Discovery/Web-Content/common.txt
First vhost subdomain enumeration:
```shellsession
gobuster vhost -u "http://fuzzing_fun.htb:30125" -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -xs 404,403,400,401 --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://fuzzing_fun.htb:30125
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
hidden.fuzzing_fun.htb:30125 Status: 200 [Size: 45]
```
When going to this subdomain in browser, it prompts us to /godeep folder. We fuzz this recursively using ffuf:
```shellsession
ffuf -u http://hidden.fuzzing_fun.htb:30125/godeep/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt --recursion                                                               

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://hidden.fuzzing_fun.htb:30125/godeep/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 33ms]
.htaccess               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 27ms]
.hta                    [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 29ms]
index.php               [Status: 200, Size: 13, Words: 2, Lines: 1, Duration: 29ms]
stoneedge               [Status: 301, Size: 352, Words: 20, Lines: 10, Duration: 21ms]
[INFO] Adding a new job to the queue: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/FUZZ

[INFO] Starting queued job on target: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/FUZZ

.hta                    [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 22ms]
.htpasswd               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 22ms]
.htaccess               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 26ms]
bbclone                 [Status: 301, Size: 360, Words: 20, Lines: 10, Duration: 19ms]
[INFO] Adding a new job to the queue: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/bbclone/FUZZ

index.php               [Status: 200, Size: 15, Words: 2, Lines: 1, Duration: 21ms]
[INFO] Starting queued job on target: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/bbclone/FUZZ

.hta                    [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 26ms]
.htaccess               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 28ms]
.htpasswd               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 29ms]
index.php               [Status: 200, Size: 18, Words: 4, Lines: 1, Duration: 24ms]
typo3                   [Status: 301, Size: 366, Words: 20, Lines: 10, Duration: 23ms]
[INFO] Adding a new job to the queue: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/bbclone/typo3/FUZZ

[INFO] Starting queued job on target: http://hidden.fuzzing_fun.htb:30125/godeep/stoneedge/bbclone/typo3/FUZZ

.htaccess               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 29ms]
.htpasswd               [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 29ms]
.hta                    [Status: 403, Size: 290, Words: 20, Lines: 10, Duration: 29ms]
index.php               [Status: 200, Size: 23, Words: 1, Lines: 1, Duration: 20ms]
:: Progress: [4750/4750] :: Job [4/4] :: 1739 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```
We found our hidden file with the flag.