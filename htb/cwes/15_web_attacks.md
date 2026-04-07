* HTTP Verb Tampering
* Insecure Direct Object References
* XML External Entity Injection

# HTTP Verb Tampering
* HTTP protocol accepts various HTTP methods as `verbs` at beginning of a request
* occurs when web server has a misconfiguration of accepted HTTP methods
* http has [9 different verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

**Insecure Configurations**
* insecure configurations cause first type of HTTP verb tampering vulns
* for example: web server's auth config may be limited to specific HTTP methods which would leave some HTTP methods accessible without authentication

**Insecure Coding**
* insecure coding causes the other type of HTTP verb tampering
* can occur when web developer applies specific filters to mitigate particular vulnerablities while not covering all HTTP methods
* For example: SQL sanitization that only happens on GET requests -> other methods might be used to bypass Sanitization

## Bypassing Basic Authentication
* relatively straightforward process
* We just need to try alternate HTTP methods to see how they are handled
* verb tampering because of insecure configurations are relatively easy to spot automatically
* 