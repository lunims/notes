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


### Proxy setup:
* Firefox preferences -> set up proxies (both listen to port 8080 on default)
* Also Foxy Proxy extensions helps when manually setting proxy
* Installing CA certificate:
	* Burp: http://burp
	* ZAP: Tools>Options>Network>Server Certificates
		* can also generate new one
	* can install in firefox under: about:preferences#privacy
		* Authorities tab -> click Import and select CA
	* 