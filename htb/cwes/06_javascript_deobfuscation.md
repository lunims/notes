* basically every website nowadays utilizes JS
* open source code of web page with [CTRL + U] (HTML) or look in Dev Tools (F12)
* JavaScript can be both inline or external (src attribute)

# Code Obfuscation
* What is obfuscation?
	* technique to make a script more difficult to read by humans while still maintaining its full functionality (performance could be slower)
	* usually done with an obfuscation tool
* Use cases:
	* hide original code
	* provide a security layer when dealing with authentication or encryption (auth and encryption should not be done on client-side anyway)
	* most common use case: malicious actions
		* common for attackers to obfuscate their malicious code to prevent Intrusion Detection
### Basic Obfuscation
* Minifying JS code
	* code minification means having entire code in a single line
	* can become quite long (line)
	* more useful for longer code 
	* Common Tool: JavaScript Minifier (https://www.toptal.com/developers/javascript-minifier)
* Packing JavaScript code
	* https://beautifytools.com/javascript-obfuscator.php
	* a packer tool usually attempts to convert all words and symbols into a list/dict and then refer to them using the (p,a,c,k,e,d) function to rebuild the original code during execution
	* this function can be different from one packer to another
### Advanced Obfuscation
* Obfuscator
	* https://obfuscator.io/
	* for example with base64 encoding
* https://jsfuck.com/
* https://utf-8.jp/public/jjencode.html
* https://utf-8.jp/public/aaencode.html

## Deobfuscation
* Beautify
	* most basic method to "beautify" code is over Browser Dev Tools
		* open script with Dev Tool and let it pretty format
		* Firefox: [CTRL+SHIFT+Z] (browser debugger) -> click on script -> click [{ }] button on bottom
		* same button for chrome 
	* Online tools
		* https://prettier.io/playground/
		* https://beautifier.io/
	* against minification
* Deobfuscate
	* https://matthewfl.com/unPacker.html
	* another way is to find the return value at the end and use console.log to print it instead of executing it
* Reverse engineering
	* once code becomes more obfuscated and encoded, it becomes more difficult
	* also custom obfuscation methods could be used
	* Reverse engineering might be needed

## Decoding
* **Base64**
	* spotting it
		* look out for padding at the end (`=` signs)
		* length of base64 has to be a multiple of 4
	* Encode: `echo https://www.hackthebox.eu/ | base64`
	* Decode: `echo aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K | base64 -d`
* **Hex**
	* encodes each character into its hex order in ASCII table
	* spotting it
		* hex characters only
		* `0-9` and `a-f`
	* Encoding: `echo https://www.hackthebox.eu/ | xxd -p`
	* Decoding: `echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r`
* **Caesar/ROT13**
	* shifts each letter by a fixed number
	* ROT13 shifts by 13
	* spotting it
		* look out for common substitutions, e.g. `http://www` becomes `uggc://jjj`
	* Encoding: `echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'`
	* Decoding: `echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'`
* Other types of encoding:
	* Determine encoding first, then look for tools
	* https://www.boxentriq.com/analysis/cipher-identifier

# Skill Asssessment

We first open the source code of the web page, which provides a link to an external JS file, which is the answer to the first question.

Then open the file and run its code in the console to get the second answer.

Furthermore, the code is obfuscated we use unpacker Online tool to deobfuscate which has the flag for the third question in its source code.

We can now build a curl request to the endpoint specified in the source code:
```shellsession
curl -X POST http://154.57.164.79:32454/keys.php -d "param1=0x437f8b"
```
This returns a weird looking value. However, we spot that its a hex value as there are only hex characters used. We can now decode this value:
```shellsession
echo "4150495f70336e5f37333537316e365f31355f66756e" | xxd -p -r
API_p3n_73571n6_15_fun 
```
With editing our previous curl command we are now able to get the flag for the last question:
```shellsession
curl -X POST http://154.57.164.79:32454/keys.php -d "key=API_p3n_73571n6_15_fun"
```
