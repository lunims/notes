* Primary goals of web reconnaissance:
	* **Identifying assets**: uncovering all accessible components of a target (webpages, subdomains...) -> comprehensive overview of target's online presence
	* **Discovering Hidden Information**: locate sensitive information (such as backup files, config file, internal docs...) -> valuable insight and potential entry points
	* **Analysing Attack Surface**: examining potential attack surface to identify potential vulnerabilities -> assess used technologies, configs and possible entry points
	* **Gathering Intelligence**: collect information that can be leveraged for further exploitation/social engineering -> identify key personnel, email address or patterns of behavior
* Two types of recon
	* Active
		* Port Scanning -> nmap, masscan, Unihornscan
		* Vulnerability Scanning -> Nessus, OpenVAS, Nikto
		* Network Mapping -> Traceroute, Nmap
		* Banner Grabbing -> Netcat, cUrl
		* OS Fingerprinting -> nmap (-O), Xprobe2
		* Service Enumeration -> nmap (-sV)
		* Web Spidering -> Burp Suite Spider, OWASP ZAP Spider, Scrapy
	* passive
		* Search engine queries -> Google, DuckDuckGo...
		* WHOIS Lookup -> whois (CLI)
		* DNS -> dig, nslookup, host, dnsenum, fierce, dnsrecon
		* Web Archive Analysis -> Wayback Machine
		* Social Media Analysis -> LinkedIn, Twitter, Facebook...
		* Code Repositories -> GitHub, GitLab

# WHOIS
* query and response protocol designed to access databases that store information about registered internet resources
* basically a phonebook for the internet
* Usage:
```shell-session
lunims@htb[/htb]$ whois inlanefreight.com

[...]
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
[...]
```
* WhoIs record holds:
	* Domain name: domain name itself (example.com)
	* Registrar: company where domain was registered
	* Registrant Contact: person/org that registered the domain
	* Administrative Contact: person responsible for managing domain
	* Technical Contact: person handling issues related to domain
	* Creation and Expiration dates: date of registration and expiration
	* Name Servers: Servers that translate domain name into IP address
### Utilizing WHOIS
* Scenario 1: Phishing Analysise.g. 
	* e.g., WHOIS could reveal, that domain was only registered a few days ago
* Scenario 2: Malware Analysis
	* e.g. domain linked to malware is registered to an individual using a free email service
* Scenario 3: Threat intelligence report
	* gather WHOIS data on multiple domains associated with group to compile a comprehensive threat intelligence report
* Installation via apt:
```shell-session
lunims@htb[/htb]$ sudo apt update
lunims@htb[/htb]$ sudo apt install whois -y
```
* Usage: `whois <domain_name>`

# DNS
1. Computer asks for direction (DNS Query): when entering domain name, computer first checks local cache to look for IP address. If not, it reaches out to DNS resolver (usually ISP)
2. DNS Resolver checks its Map (Recursive Lookup): resolver also has cache. If no IP address is found, it starts iterating over DNS hierarchy, starting with a root name server
3. Root Name Server points the way: root server does not know the exact address but knows who does -> points to TLD name server (.com, .org)
4. TLD Name Server Narrows it down: TLD name server is like a regional map. It knows which authoritative name server is responsible for the specific domain you're looking for 
5. Authoritative Name Server delivers address: authoritative name server is the final stop. It holds correct IP address and sends it back to resolver
6. The DNS Resolver returns Information: DNS Resolver receives IP and sends it back to the computer (also caches it)
7. Your Computer connects: The computer now knows the IP address and can connect. It also caches the IP
* Hosts File:
	* Linux: `/etc/hosts`
	* Windows: `C:\Windows\System32\drivers\etc\hosts`
* **Key DNS Concepts**
	* **Zone**:
		* distinct part of domain namespace that a specific entity/adminsitrator manages
		* like a virtual container for a set of domain names
		* Zone file (residing on DNS server) defines the resource records wihtin this zone
	* **Resource Records**:
		* Domain Name: example.com
		* IP-address: 192.0.0.1
		* DNS Resolver: 8.8.8.8
		* Root Name Server: a.root-servers.net
		* TLD Name Server: [Verisign](https://en.wikipedia.org/wiki/Verisign) for `.com`, [PIR](https://en.wikipedia.org/wiki/Public_Interest_Registry) for `.org`
		* Authoritative Name Server: Often managed by hosting providers or domain registrars.
		* DNS Record Types: 
			* A: IPv4 addr
			* AAAA: IPv6 addr
			* CNAME: Canonical Name Record -> creates alias for a hostname pointing to another hostname
			* MX: Mail Exchange Record -> mail server(s) responsible for handling email for the domain
			* NS: Name Server Record -> delegates a DNS Zone to a specific authoritative name server
			* TXT: Text Record -> stores arbitrary text information (often used for domain verification or security policies)
			* SOA: Start of Authority Record -> Specifies administrative information about a DNS zone, including the primary name server, responsible person's email, and other parameters.
			* SRV: Service Record -> defines hostname and port number for specific services
			* PTR: Pointer Record -> used for reverse DNS Lookup, mapping an IP address to a hostname
			* IN: Internet -> In essence, "`IN`" is simply a convention that indicates that the record applies to the standard internet protocols we use today
### Digging DNS
* DNS Tools:
	* dig
	* nslookup
	* host
	* dnsenum
	* fierce
	* dnsrecon
	* theHarvester
* **dig**
	* `dig domain.com` -> performs default A record lookup for domain
	* `dig domain.com A` -> retrieves IPv4 for domain
	* `dig domain.com AAAA` -> retrieves IPv6
	* `dig domain.com MX` -> finds mail servers for domain
	* `dig domain.com NS` -> finds authoritative name servers for domain
	* `dig domain.com TXT` -> retrieves TXT records for domain
	* `dig domain.com CNAME` -> retrieves Canonical Name record for domain
	* `dig domain.com SOA` -> retrieves the start of authority
	* `dig @1.1.1.1 domain.com` -> specifies a specific name server to query
	* `dig +trace domain.com` -> shows full path of DNS resolution
	* `dig -x 192.168.1.1` -> reverse lookup for provided IP addr
	* `dig +short domain.com` -> provides short, concise answer to query
	* `dig +noall +answer domain.com` -> Displays only the answer section of the query output.
	* `dig domain.com ANY` -> retrieves all available DNS records for the domain
* Header: 
	* `;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16449`
		* the type of Query (QUERY), the successful status (NOERROR) and a unique identifier (16449) for this specific querty
			* `;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0`
				* qr: query response flag - indicates this is a response 
				* rd: Recursiom Desired flag - means recursion was requested
				* ad: Authentic Data flag - means the resolver considers the data authentic
				* The remaining numbers indicate the number of entries in each section of the DNS response: 1 question, 1 answer, 0 authority records, and 0 additional records
			* `;; WARNING: recursion requested but not available`: indicates that recursion was requested, but server does not support it
* Question Section:
	* `;google.com. IN A` -> this line specifies question "What is the IPv4 address (A) for google.com"
* Answer Section: 
	* `google.com. 0 IN A 142.251.47.142` -> answer to the query. 0 is TTL
* Footer:
	* `;; Query time: 0 msec` -> time it took for the query 
	* `;; SERVER: 172.23.176.1#53(172.23.176.1) (UDP)` -> identifies the DNS server that provided the answer and the protocol used (UDP)
	* `;; WHEN: Thu Jun 13 10:45:58 SAST 2024` -> timestamp of the query
	* `;; MSG SIZE rcvd: 54` -> msg size

### Subdomains
* Subdomain enumeration: Process of systemically identifying and listing subdomains
* subdomains typically represented as A or AAAA records which map the subdomain to the IP
* CNAME might be used to create aliases
* Active Subdomain enumeration
	* directly interacting  with the target domain's DNS server to uncover subdomains
	* DNS Zone transfer: a misconfigured server might inadvertently leak a complete list of of subdomains (rarely successful)
	* brute-force enumeration: systemically testing a list of potential subdomains (tools: dnsenum, ffuf, gobuster)
* Passive Subdomain enumeration:
	* relies on external resources of information to discover subdomains without directly querying the target's DNS servers
	* valuable resource: Certificate Transparency (CT) logs
	* search engines

### Subdomain Bruteforcing
1. Wordlist Selection: 
	1. General-Purpose: broad range of subdomain names (e.g. dev, admin, test). Approach useful when no info about target's naming conventions
	2. Targeted: focused on specific industries, technologies or naming patterns relevant to target
	3. Custom: create own wordlists
2. Iteration and Quering: Script/Tool iterates over wordlist and tries to create potential subdomain names
3. DNS Lookup: DNS query is performed for each potential subdomain
4. Filtering and Validation: if subdomain resolves successfully, it's added to the list of valid subdomains
* Tools:
	* dnsenum
	* fierce
	* dnsrecon
	* amass
	* assetfinder
	* puredns
* **dnsenum**
	* DNS Record Enumeration: can retrieve various DNS records, providing comprehensive overview of the target's DNS configuration
	* Zone Transfer Attempts: tool automatically attempts zone transfers from discovered name servers.
	* Subdomain brute-forcing
	* Google Scraping: scrapes Google search results to find additional subdomains
	* Reverse Lookup: reverse DNS lookups to identify domains associated with given IP addr
	* WHOIS lookups
* Subdomain brute-forcing with wordlist:
```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```
* --enum: specifies target domain
* -f: indicates path to wordlist
* -r: enables recursive subdomain brute-forcing

### DNS Zone Transfers
* What is a Zone transfer?
	* essentially a wholesale copy all DNS records within a zone (a domain and its subdomains)
	* essential for maintaining consistency and redundancy across DNS servers
	* if not adequately secured: unauthorised parties can download entire file zone, revealing a full list of subdomains
		1. Zone Transfer Request (AXFR): secondary DNS server requests zone transfer (AXFR type)
		2. SOA Record Transfer: primary server responding by sending its Start of Authority record (SOA) record. SOA contains vital information such as serial number
		3. DNS Record Transmission: primary server then transfers all the DNS records in the zone to secondary server, one by one. Includes records like A, AAAA...
		4. Zone Transfer complete: once all records have been transmitted, primary server signals the end of the zone transfer
		5. Acknowledgement (ACK): secondary server sends an ackowledgement
* should not be accessible by anyone
* Exploiting Zone Transfers (if misconfigured)
```shell-session
lunims@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me
```

### Virtual Hosts
* ability of web servers to distinguish between multiple websites/apps sharing the same IP address
* achieved over HTTP host header
* difference to subdomains:
	* subdomains: extension of domain name (e.g. blog.example.com), typically have their own DNS records, pointing either to same or different IP address
	* VHosts: Virtual hosts are configurations within a web server that allow multiple websites or applications to be hosted on a single server. They can be associated with top-level domains or subdomains. Each virtual host can have its own separate configuration, enabling precise control over how requests are handled
* if virtual host does not have DNS record -> still accessable via modification of hosts file
* VHost fuzzing: technique to discover public and non-public subdomains by testing various hostname
* Server VHost lookup
	1. Browser requests a website
	2. Host Header reveals the domain
	3. Web server determines the virtual host
	4. Serving the right content
* Types of virtual hosting:
	* Name-based Virtual Hosting: method relies on the HTTP Host header
	* IP-Based Virtual Hosting: assign unique IP address to each website hosted on the server, server knows which website to server based on IP
	* Port-Based Virtual Hosting: based on port
* Virtual Host Discovery Tools:
	* gobuster
	* Feroxbuster
	* ffuf
* gobuster
	* versatile tool used for directory and file brute-forcing
```shell-session
lunims@htb[/htb]$ gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```
* -u URL
* -w wordlist
* --append-domain flag appends base domain to each word in wordlist
* -t can increase the used threads
* -k flag ignores SSL/SSL certificate errors
* -o flag to save output to a file

### Certificate Transparency
* public servers that log the issuance of a certificate
* purposes
	* early detection of rogue certificates
	* accountability for Certificate Authorities
	* Strenghtening the Web PKI (Public Key Infrastructure)
* tools for recon:
	* crt.sh (web interface)
	* Censys
* crt.sh via curl
```shell-session
lunims@htb[/htb]$ curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
 
*.dev.facebook.com
*.newdev.facebook.com
*.secure.dev.facebook.com
dev.facebook.com
devvm1958.ftw3.facebook.com
facebook-amex-dev.facebook.com
facebook-amex-sign-enc-dev.facebook.com
newdev.facebook.com
secure.dev.facebook.com
```
* `jq -r '.[] | select(.name_value | contains("dev")) | .name_value'`: filters JSON results, selecting only entries where the name_value field (domain/subdomain) includes string "dev"
* `sort -u`: sorts results alphabetically and removes duplicates

# Fingerprinting
* focuses on extracting technical details about technologies powering a website/app
* digital signature of web servers, OSes and software components can reveal critical information about a target's infrastructure
* purposes:
	* targeted attacks
	* identifying misconfigurations
	* priorising targets
	* building a comprehensive profile
* techniques:
	* Banner Grabbing: involves analysing the banners presented by web servers and other services
	* Analysing HTTP headers: headers contain a wealth of information (Server header typically discloses web server software, X-Powered-By header might reveal additional technologies like scripting languages/framework)
	* Probing for specific reasons: sending specifically crafted requests to the target can generate unique responses that reveal technologies (for example error messages)
	* Analysing page content: web's page content, including its structure, scripts and other elements can often provide clues about underlying technologies (e.g. copyright header)
* Tools:
	* Wappalyzer (Browser Extension)
	* BuiltWith (Web technology profiler)
	* WhatWeb (CLI)
	* Nmap (CLI)
	* Netcraft 
	* wafw00f (CLI) 
* grab banner HTTP headers via curl 
```shell-session
lunims@htb[/htb]$ curl -I https://www.inlanefreight.com

HTTP/1.1 200 OK
Date: Fri, 31 May 2024 12:12:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8
```
* identify if Web Application is used via wafw00f
```shell-session
lunims@htb[/htb]$ pip3 install git+https://github.com/EnableSecurity/wafw00f
```
```shell-session
lunims@htb[/htb]$ wafw00f inlanefreight.com

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://inlanefreight.com
[+] The site https://inlanefreight.com is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2
```
* nikto also offer fingerprinting tools
```shell-session
lunims@htb[/htb]$ nikto -h inlanefreight.com -Tuning b
```
* -h specifies host
* -Tuning b tells nikto to run Software identification modules

# Crawling
* automated process of systemically browsing World Wide Web
* web crawler follows links from one page to another, collecting information
* How Web Crawlers work:
	* starts with seed URL
	* fetches pages, parses content and extracts all links
* Breadt-First Crawling: Prioritizes exploring a website's width before going deep
* Depth First Crawling: Prioritizes exploring a website's death before going broad
* Extracting Valuable Information:
	* Links (Internal and External)
	* Comments
	* Metadata (data about data)
	* Sensitive files
### robots.txt
* acts as a virtual "etiquette guide" for bots, outlining which areas of a website they are allowed to access and which are off-limits
* simple text file places in root directory of a website
* Directives:
```txt
User-agent: *
Disallow: /private/
```
* this tells all user-agents (* wildcard) that they are not allowed to access any URLs that start with /private/
* User-agent: this line specifies which crawler or bot the following rules apply to
* Directive: 
	* Disallow: specify paths that bot should not crawl
	* Allow: Specify paths that bot is allowed to crawl (even if they fall under a broader Disallow rule)
	* Crawl-delay: sets a delay (in seconds) between successive requests from the bot to avoid overloading the server
	* Sitemap: provides the URl to an XML sitemap for more efficient crawling
* Why respect robots.txt?
	* an evil crawler could just ignore it
	* avoiding overburdance of server
	* protecting sensitive information
	* legal and ethical compliance
* robots.txt in Web Recon:
	* Uncovering Hidden directories
	* Mapping Website Structure
	* Detecting crawler types

### Well-known URIs
* `.well-known` standard serves as a standardized directory within a website's root domain
* tpyically accessible with `/.well-known/` path on web server
* https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml
* examples:
	* security.txt -> contains contact information for sec related stuff
	* /.well-known/change-password -> provides a standard URL to password change
	* openid-configuration -> config detail for OpenID Connect
	* assetlinks.json -> used for verifying ownership of digital assets
	* mta-sts.txt -> specifies policy for SMTP MTA Strict Transport Security to enhance email security
* web recon:
	* particularly useful: openid-configuration
		* openid identity layer built on top of Oauth 2.0
	* contains multiple exploration opportunitites:
		* Endpoint discovery:
			* Authorization Endpoint
			* token endpoint
			* userinfo endpoint
		* JWKS URI: `jwks_uri` reveals the `JSON Web Key Set` (`JWKS`), detailing the cryptographic keys used by the server
		* supported scopes and response types: Understanding which scopes and response types are supported helps in mapping out the functionality and limitations of the OpenID Connect implementation
		* algorithm details: Information about supported signing algorithms can be crucial for understanding the security measures in place

### Creepy Crawls
* Popular Web Crawlers:
	* Burp Suite Spider
	* OWASP ZAP
	* Scrapy (Python Framework)
	* Apache Nutch (Scalable Crawler)
### Scrapy:
`pip3 install scrapy`
* Recon Spider:
```shellsession
lunims@htb[/htb]$ wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip lunims@htb[/htb]$ unzip ReconSpider.zip
```
Also available in kali repos (sudo apt install reconspider)
Run it via:
```shellsession
lunims@htb[/htb]$ python3 ReconSpider.py http://inlanefreight.com
```
* results.json
	* after ReconSpider.py, results will be sent to a JSON file (results.json)
	* includes found emails, links, external_files...
	* keys:
		* emails
		* links
		* external_files
		* js_files
		* form_fields
		* images
		* videos
		* audio
		* comments
		* 

# Search Engine Discovery
* Open Source Intelligence (OSINT) gathering
* using search engines for intelligence gathering
* Search Operators:
	* site: -> Limits results to a specific website or domain
	* inurl: -> finds pages with a specific term in the URL
	* filetype: -> searches for a particular file type
	* intitle: -> finds pages with specific term in the title
	* intext: / inbody: -> searches for a term within body text
	* cache: -> displays the cached version of a webpage
	* link: -> finds pages that link to a specific webpage
	* related: -> finds websites related to a specific webpage
	* ...
* Google Dorking:
	* leverages the power of search operators to uncover sensitive information, security vulnerabilities or hidden content on websites using Google search
	* Examples:
		* Finding login pages:
			* - `site:example.com inurl:login`
			* - `site:example.com (inurl:login OR inurl:admin)`
		* Identifying exposed files:
			* - `site:example.com filetype:pdf`
			* - - `site:example.com (filetype:xls OR filetype:docx)`

# Web Archives
* Wayback Machine
* digital archive of World Wide Web
* uses web crawlers to capture snapshots of websites at regular intervals
* 3 step process: Crawling -> Archiving -> Accessing
* Recon:
	* Uncovering hidden assets and vulnerabilities (old ones)
	* Tracking Changes and Identifying patterns
	* Gathering Intelligence
	* Stealthy Reconnaissance

#  Automating Recon
* Recon Frameworks:
	* FinalRecon 
	* Recon-ng
	* theHarvester
	* SpiderFoot
	* OSINT Framework
### FinalRecon
* information:
	* Header Information
	* Whois Lookup
	* SSL Certificate Information
	* Crawler
	* DNS Enumeration
	* Subdomain Enumeration
	* Directory Enumeration
	* Wayback machine
```shellsession
lunims@htb[/htb]$ git clone https://github.com/thewhiteh4t/FinalRecon.git lunims@htb[/htb]$ cd FinalRecon lunims@htb[/htb]$ pip3 install -r requirements.txt
lunims@htb[/htb]$ chmod +x ./finalrecon.py 
lunims@htb[/htb]$ ./finalrecon.py --help 
```
* Options
	* --url target URL
	* --headers retrieve header information
	* --sslinfo get ssl cert info
	* --whois perform WhoIs lookup
	* --crawl Crawl target
	* --dns perform dns enumeration
	* --sub enumerate subdomains
	* --dir search for directories
	* --wayback retrieve wayback URLs
	* --ps perform fast port scan
	* --full perform a full recon scan on target

# Skill Assessments
* Scrapy:
```shellsession
pip3 install scrapy  
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip  
unzip ReconSpider.zip  
python3 ReconSpider.py http://inlanefreight.htb:34677  
cat results.json | jq ".comments"
```

### Final
1. Query Whois for inlanefreight
```
whois inlanefreight.com
```

2. Look at Server sent server header:
```
curl -I 154.57.164.73:30641
```
3. Enumerate VHosts first:
```
gobuster vhost -u http://inlanefreight.htb:32767 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://inlanefreight.htb:32767
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
#www.inlanefreight.htb:32767 Status: 400 [Size: 157]
#mail.inlanefreight.htb:32767 Status: 400 [Size: 157]
#smtp.inlanefreight.htb:32767 Status: 400 [Size: 157]
#pop3.inlanefreight.htb:32767 Status: 400 [Size: 157]
web1337.inlanefreight.htb:32767 Status: 200 [Size: 104]
Progress: 114442 / 114442 (100.00%)
===============================================================
Finished
===============================================================

```
Add new found subdomain to /etc/hosts
When looking at robots.txt, we find directory admin_h1dd3n. Enumerate this with gobuster:
```
└─$ gobuster dir -u http://web1337.inlanefreight.htb:32767/admin_h1dd3n/ -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://web1337.inlanefreight.htb:32767/admin_h1dd3n/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 255]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================

```
And we found index.html, where we will find the answer to question 3.

4. Since web1337.inlanefreight.htb is a dead-end crawling wise, let's look further for subdomains here:
```
└─$ gobuster vhost -u http://web1337.inlanefreight.htb:32767 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://web1337.inlanefreight.htb:32767
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
dev.web1337.inlanefreight.htb:32767 Status: 200 [Size: 123]
```
We find a dev. subdomain which we can use ReconSpider on.
In results.json we can find the email
```
cat results.json | jq ".emails"
```
5. Answer:
```
cat results.json | jq ".comments"
```