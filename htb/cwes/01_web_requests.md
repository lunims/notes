# HTTP Fundamentals
## HyperText Transfer Protocol

* application level
* client-server architecture
* default port: 80 
* Fully **Qualified Domain Name (FQDN)** as **Uniform Resource Locator (URL)**

### URL

Components:
* Scheme: http:// or https:// => identifies used protocol
* User Info: admin:password => optional user:password value for authentication
* Host: example.com => host signifies resource location, hostname or IP address
* Port: :80 => port is separated by a column, default to 80 if none specified
* Path: dashboard.php => points to the resource being accessed
* Query String: ?login=true => starts with question mark and can contain multiple params separated with &
* Fragments: #status => Fragments are only processed by browser and not sent to the server. used to locate sections on client-side


### HTTP flow
![[http_flow.png.png]]

1. request to DNS server (first time a user visits a URL)
2. DNS server responds with IP
3. browser sends GET request for the root path **/** 
4. by default: web servers are configured to return an index file when **/** is retrieved
5. the web browser receives (with HTTP status code 200) and renders **index.html**

## HyperText Transfer Protocol Secure (HTTPS)

* HTTP is unencrypted
* prone to packet sniffing (network attacker)
* Hence HTTPS: encrypted data
* default port: 443
* Note: the visited URL might still be revealed if a clear-text DNS server is contacted

![[https_flow.png.png]]
* http request sent to server
* server enforces https so sends a redirect to https with default port **443**
* **Client hello** send to server: revealing some information 
* Server responds with **server hello**
* Then **key-exchange** with **SSL-certificates**
* HTTP encrypted communication 


## HTTP Requests and Responses

* HTTP communication consits of: **requests** and **responses**
* request **sent** by client, **processed** by server
* once request has been process, server sends a **HTTP response** with **status code** and (maybe) the **requested data**

### Requests

![[http_request.png.png]]

* Method: GET => specifies type of action to perform
* Path: /users/login.html => path to resource being acessed, can also contain querys with ? and &
* Version: HTTP/1.1 => denotes the used HTTP version
* HTTP headers afterwards

### Response

![[http_response.png.png]]
* first line: **HTTP version** + **HTTP status code**
* followed by headers
* followed by body with (maybe) requested resource: can be html, json...

### Browser Dev Tools
* browsers have built-in developer tools 
* opened via: [`CTRL+SHIFT+I`] or [`F12`]

## HTTP Headers
* some headers are unique to request or response, some can be present in both
* can have one or multiple values (separated with semicolon)
* Five headers categories:
	* General Headers
	* Entity Headers
	* Request Headers
	* Response Headers
	* Security Headers

### General Headers
* Date: holds date and time at which msg originated
* Connection: possible values close or keep-alive

### Entity Headers
* Content-Type: describes type of transferred resource
* Media-type: similar to Content-type, describes data being transferred
* Boundary: boundary="b4e4fbd93540", acts as a marker to seperate content when there is more than one in the message
* Content-Lenght: length of the message
* Content-Encoding: type of encoding if message is encoded, for example gzip

### Request Headers
* Host: specifies host being queried for the resource -> good enumeration target
* User-Agent: describes client being used 
* Referer: denotes where the request is coming from
* Accept: describes which media type the client can understand, \*/\* -> accept all media types
* Authorization: used for authentication, for example base64 encoded user:pass values

### Response Headers
* Server: contains information about server, that processed message
* Set-Cookie: browsers parse this and store attached cookie
* WWW-authenticate: notifies client about authentication type being used

## Security Headers
* Content-Security-Policy: dictates the website's policy towards externally injected resources
* Strict-Transport-Policy: prevents browsers from accessing HTTP protocol and forces HTTPS
* Referrer-Policy: dicates whether browser should include the value of Referer or not

A complete list of HTTP headers can be found here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers

## cUrl:
* Basic usage: curl inlinefright.com
* The certificate check can be skipped with **-k** or **--insecure**
* Download with - O flag: curl -O inlinefright.com
* **-I** sends a Head request, so that only the response headers can be seen
* full previes of HTTP request with **verbose flag: -v


# HTTP Methods

## HTTP methods and status codes

* several request methods are allowed to send information, forms or files to the server

### HTTP Request
* **GET**: request a specific resource, can contain query parameter via ?
* **POST**: send data to the server, data is in message body
* **HEAD**: only request response header if GET was sent 
* **PUT**: creates a new resource on the server
* **DELETE**: delete an existing resource 
* **OPTIONS**: returns information about the server, such as methods accepted by it (preflight for CORS)
* **PATCH**: applies partial modifications to the resource at specified location
Note: mostly GET and POST used in practice, PUT and DELETE for REST APIs as well

### Status codes
* **1XX**: provides information and does not affect processing of the request
* **2XX**: success 
* **3XX**: redirection
* **4XX**: client-side error
* **5XX**: server-side error
Full list: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

## GET
* sent when visiting a URL
### HTTP basic auth
* server prompts to do a authentication with 401 status code
* User gives user:pass value
* Server validates, either 2XX or 403 Forbidden
* **-U flag** in curl to provide user:pass value
### Authorization header
* basic auth: **Authorization: Basic <basic64 encoded user:pass**
* Bearer: more modern types of authentication like **JWT**
* most web apps use login forms with POST requests (in combination with cookies) for authentication

### GET parameters
* parameter attached to GET request via ?
* for example a search parameter ?search=foo

## POST
* send data to the server
* places in request body
	* lack of logging: may contain lots of data and would be inefficient
	* less encoding requirements: URLs are designed to be shared, so they need to conform to characters that can be turned into letters
	* more data can be sent: for URLs there is a maximum length
### Login forms
* POST requests sent via form instead of HTTP Auth 
* **-X flag** in cUrl to specify request type 
* **-d flag** in cUrl to send data
### Authenticated Cookies
* if authentication successful -> server sends cookie
* **-b flag** in cUrl to set cookie
* cookies can also bet set in storage tab in Dev Tools

## CRUD API
### API
* Application Programming Interface
* many APIs are used to interact with databases
	* specify requested table/row within API query and then use HTTP method to determine operation
* **CRUD:**
	* Create: POST -> add data to the database
	* Read: GET -> read data from the database
	* Update: PUT -> update data from the database
	* Delete: DELETE -> delete data from the database
* CRUD principle also often used REST APIs
* useful for cUrl: pipe cUrl output into **jq** utility, so that is readable -> *curl ... | jq*
* **Update**
	* PUT: we need to specify the name of entity we want to change in URL
	* data put into message body
* **Delete**:
	* simply specify the name to be deleted in URL

