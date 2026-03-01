## Introduction 
* client-server architecture
* front-end: what the user sees
* back-end: server-side
* Web applications vs. web sites
	* static pages are often referred as Web 1.0
	* web applications (dynamic) are often referred as Web 2.0
	* Furthermore web applications:
		* being modular
		* running on any display size
		* running on any platform without being optimized
* Web applications vs native code:
	* web apps are platform independant and can run in any browser
	* do not have to be installed on the user's system
	* version unity
	* slower performance
* Web application distribution
	* common open source web apps include:
		* WordPress
		* OpenCart
		* Joomla
	* proprietary "closed source":
		* Wix 
		* Shopify
		* DotNetNuke
* Security risks of web apps:
	* usually accessible by anyone
	* succesful attack can lead to significant losses and business interruptions
	* often data leakages or database leakage
	* OWASP Web App Testing Guide: https://github.com/OWASP/wstg/tree/master/document/4-Web_Application_Security_Testing
* Attacking web apps:
	* SQL Injection -> obtaining Active Directory username and performing a password spraying attack against a VPN or email portal
	* File Inclusion -> reading source code to find a hidden page which exposes additional functionality that can be used to gain RCE
	* Unrestricted File Upload -> A web application that allows a user to upload a profile picture that allows any file type to be uploaded (not just images). This can be leveraged to gain full control of the web application server by uploading malicious code.
	* Insecure Direct Object Referencing (IDOR) -> When combined with a flaw such as broken access control, this can often be used to access another user's files or functionality. An example would be editing your user profile browsing to a page such as /user/701/edit-profile. If we can change the `701` to `702`, we may edit another user's profile!
	* Broken Access Control -> Another example is an application that allows a user to register a new account. If the account registration functionality is designed poorly, a user may perform privilege escalation when registering. Consider the `POST` request when registering a new user, which submits the data `username=bjones&password=Welcome1&email=bjones@inlanefreight.local&roleid=3`. What if we can manipulate the `roleid` parameter and change it to `0` or `1`. We have seen real-world applications where this was the case, and it was possible to quickly register an admin user and access many unintended features of the web application.

## Web Application Layour
* Web Applications Infrastructure -> describes the structure of required components, such as the database, needed for the web app to function. Since web app can be set up on a separate server, it is essential to know which database server it needs to access
* Web Application Components -> the components make up a web app represent all the components that the web app interacts with. These are divided into the following areas: `UI/UX`, `Client`, and `Server` components.
* Web application architecture: architecture compromises all the relationship between the various web app components

### Web Application Infrastructure
* Client-Server
	* server hosts web app and distributes it to any client accessing it 
* one Server
	* entire web app or several web apps and their components including database are all hosted on the a single server (riskiest design)
* Many servers - one database
	* separates database on its own database server (segmentation)
* Many servers - many databases
	* each web application's data is hosted in a separate database (also for redundancy purposes)

### Web App components
* Client
* Server
* Services (Microservices)
	* 3rd Party integrations
	* Web App Integrations
	* Microservices:
		* Independant components of the web app, for example:
			* registration
			* Search
			* ...
		* easier scaling, since they can be written in different languages and still interact
			* Agility
			* Flexible scaling
			* Easy deployment
			* Reusable code
			* Resilienc
			* 
* Functions (Serverless)
	* Serverless:
		* Cloud providers such as AWS, GCP, Azure, among others, offer serverless architectures
		* platforms provide application frameworks to build such web applications without having to worry about the servers themselves
		* web apps run in stateless computing containers (Docker)

### Web App architecture
* Presenation layer -> conists of UI process components that enable communication with the app and system. can be accessed by the client via the web browser and are returned in the form of HTML, JS and CSS
* Application layer -> This layer ensures that all client requests (web requests) are correctly processed. Various criteria are checked, such as authorization, privileges, and data passed on to the client.
* Data layer -> the data works closely with the application layer to determine where the required data is stored and can be accessed

## Front-end vs back-end
* Front-end
	* rendered on browser (client-side)
	* HTML, CSS and JS
	* should work on every device and adapt
	* https://html-css-js.com/
* back-end
	* server-side
	* platform-specific
	* 4 main components:
		* Back end server -> The hardware and operating system that hosts all other components and are usually run on operating systems like `Linux`, `Windows`, or using `Containers`
		* Web Servers -> Web servers handle HTTP requests and connections. Some examples are `Apache`, `NGINX`, and `IIS`
		* Databases -> Databases (`DBs`) store and retrieve the web application data. Some examples of relational databases are `MySQL`, `MSSQL`, `Oracle`, `PostgreSQL`, while examples of non-relational databases include `NoSQL` and `MongoDB`.
		* Development frameworks -> Development Frameworks are used to develop the core Web Application. Some well-known frameworks include `Laravel` (`PHP`), `ASP.NET` (`C#`), `Spring` (`Java`), `Django` (`Python`), and `Express` (`NodeJS JavaScript`).
	* Components can be hosted in different containers
* Securing front- and back-end
	* Client-side with source code (whitebox)
	* back-end usually black-box
	* OWASP Top 10: https://owasp.org/www-project-top-ten/

## HTML
* tree form
* document
	* head
		* title
	* body
		* h1
		* p
* each element can contain other elements (all document elements should be within the \<html> tags)
* URL Encoding
	* URLs only alphanumerical -> hence encoding for non-alphanumerical
	* Encoding table here: https://www.w3schools.com/tags/ref_urlencode.ASP
* \<head>
	* contains elements that are not directly printed on page
	* e.g. page title
* \<body> 
	* main page elements
* \<style>
	* CSS code
	* inline or external
	* styling the page
* \<script>
	* holds JS code of page
	* inline or external
* DOM (Document Object Model)
	* "The W3C Document Object Model (DOM) is a platform and language-neutral interface that allows programs and scripts to dynamically access and update the content, structure, and style of a document."
	* Core DOM:
		* standard model for all document types
	* XML DOM:
		* standard model for XML docs
	* HTML DOM:
		* standard model for HTML docs
	* For example, we can refer to elements like document.head or document.h1

## Cascading Style Sheets (CSS)
* styles the HTML elements
* Example snippet:
```css
body {
  background-color: black;
}

h1 {
  color: white;
  text-align: center;
}

p {
  font-family: helvetica;
  font-size: 10px;
}
```
* Syntax:
	* defines style of each HTML elements between curly brackets {}, within properties are defined with their values -> `element { property : value; }`
	* Example properties are: `height`, `position`, `border`, `margin`, `padding`, `color`, `text-align`, `font-size``
	* can also be used for animations
* Framework:
	* Bootstrap
	* SASS
	* Foundation
	* Bulma
	* Pure

## JavaScript (JS)
* Mostly front-end, but also some back-end JS Execution environments like NodeJS
* Within page: JS is loaded via script tags \<script>\</script>
* can be external as well: \<script src="./script.js">\</script>
* example JS:  document.getElementById("button1").innerHTML = "Changed Text!";
	* changes content of button1 html elements 
* https://jsfiddle.net/
* Usage:
	* most comon web apps use JS to drive all needed functionality on web page, like updating web page view in real-time, processing user input....
	* also used to automate complex processes and perform HTTP requests to interact with the back end components and send/retrieve data (through technologies like AJAX)
	* Also used for animations, that CSS alone can not do
	* all modern browsers are equipped with JS engines
* Frameworks:
	* Angular
	* React
	* Vue
	* jQuery

# Front End Vulnerabilities
* attacking front-end components does not pose a direct threat to back-end components
* executed on client-side -> puts the end user in danger
## Sensitive Data Exposure
* refers to the availability of sensitive data in clear-text to the end-user
* usually found in the source code of page
* Open page source code with `CTRL + U`
* Sometime data likes hashes or credentials is exposed within code
* Should be one of the first things to check out (identify low-hanging fruits)
* Look out for HTML comments: <!-- --> -> often contains information about the system

## HTML Injection
* user input should always be validated or sanitized
* should happen both in front- and back-end
	* sometimes front-end input is not sent to back-end
* HTML Injection occurs when unfiltered input is displayed on the page
	* browser may display it as part of the page
* Another form: Web page defacing -> injecting new HTML code to change web's appearence (for example load malicious apps)
* Example for vulnerable JS code snippet:
   `document.getElementById("output").innerHTML = "Your name is " + input;`
	* just concatenating the user input

## Cross-Site Scripting (XSS)
* HTML Injection can often be used to also perform Cross-Site Scripting
* inject JS code to be executed on the client-side
* **Reflected XSS** 
	* Occurs when user input is displayed on the page after processing (e.g., search result or error message).
* **Stored XSS**
	* Occurs when user input is stored in the back end database and then displayed upon retrieval (e.g., posts or comments).
* **DOM XSS**
	* Occurs when user input is directly shown in the browser and is written to an `HTML` DOM object (e.g., vulnerable username or page title)

## Cross-Site Request Forgery (CSRF)
* CSRF attacks may utilize XSS to perform certain queries that the victim is authenticated to
* once victim views payload on a vulnerable page, JS code would execute automatically
* can also be leveraged to attack admins and gain access over their accounts
* Prevention:
	* **Sanitazation**: Removing special characters and non-standard characters from user input before displaying it or storing it.
	* **Validation**: Ensuring that submitted user input matches the expected format (i.e., submitted email matched email format)
	* Another solution: Web Application Firewalls (WAF)
	* Other defenses:
		* CSRF tokens
		* SameSite cookies (set to lax or strict)
	* Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html

# Back End Components
## Back End Servers
* back-end server is the hardware and OS on the back end that hosts all of the applications necessary to run the web app
* would fit into the data access layer
* back end server contains the other 3 back end components
	* Web Server
	* Database
	* Development Framework
* Many popular combinations of "stacks" for back-end servers
	* LAMP -> Linux, Apache, MySQL and PHP
	* WAMP -> Windows, Apache, MySQL and PHP
	* WINS -> Windows, IIS, .NET and SQL Server
	* MAMP -> macOS, Apache, MySQL and PHP
	* XAMPP -> Cross-Platform, Apache, MySQL and PHP/Perl
	* https://en.wikipedia.org/wiki/Solution_stack

## Web Servers
* handles all the HTTP traffic from client-side browser, routes it to the requested page and answers to the client
* usually run on port 80 (HTTP) or 443 (HTTPS)
* Workflow:
	* accepts HTTP request from client
	* respond with HTTP response and code (like 200 OK or 404 Not Found)
	* also accept various type of user input including text, JSON and even binary data (file uploads)
* Apache (or httpd):
	* most common web server on the internet (more than 40%)
	* usually pre-installed in most Linux distros
	* Mostly used with PHP, but also .NET, Python, Perl and even OS languages like Bash through CGI
	* Apache modules can be installed to extend functionality
* NGINX:
	* second most common web server (~30%)
	* focuses on serving many concurrent web requests with relatively low memory and CPU
* IIS
	* Internet Information Services (~15%)
	* Microsoft
	* usually with .NET, but also PHP or host other services like FTP

## Databases
* used to store various content and information related to the web app
* Relational (SQL):
	* store data in tables, rows and columns
	* each table can have unique keys which can link tables together and create relationships between tables
	* For example: an retrieve all details linked to a certain user from all tables with a single query -> relational databases very fast and reliable for big datasets that have clear structure and design
	* Most common:
		* MySQL -> open source
		* MSSQL -> Microsofts implementation
		* Orcale -> reliable database for big businesses, can be costly
		* PostgreSQL -> free and open-source, designed to be easily extensible
* Non-relational (NoSQL):
	* does not use tables
	* instead stores data using various storage models, depending on the type of data stored
	* 4 common models:
		* Key-Value
		* Document-based
		* Wide-column
		* Graph
	* For example, Key-value usually stores data in JSON or XML and has a key for each pair, storing all of its data as its value
	* Document-Based model stores data in complex JSON objects and each object has certain meta-data while storing the rest of the data similarly to Key-value
	* Most common NoSQL Databases:
		* MongoDB -> Document-based
		* ElasticSearch -> open-source, free, optimized for storing and analyzing huge datasets
		* Apache Cassandra -> free and open source, scalable and optimized for handling faulty values
* Use in Web Apps:
	* modern web app dev languages makes it easy to integrate, store and retrieve data from database
		* PHP: 
Connect to database server
```php
			$conn = new mysqli("localhost", "user", "pass");
```
create a database:
```php
$sql = "CREATE DATABASE database1";
$conn->query($sql)
```
use the database:
```php
$conn = new mysqli("localhost", "user", "pass", "database1");
$query = "select * from table_1";
$result = $conn->query($query);
```
Search for data with user input:
```php
$searchInput =  $_POST['findUser'];
$query = "select * from users where name like '%$searchInput%'";
$result = $conn->query($query);
```

## Development Frameworks & APIs
* Laravel (PHP)
* Express (Node.JS) 
* Django (Python)
* Rails (Ruby)
* **APIs**
	* front end components needs to interact with back-end components
	* Query Parameters:
		* default method for sending arguments to is GET and POST
		* GET parameters in query over ?
		* POST parameters in body
* Web API:
	* for web apps, APIs is what allows remote access to functionality on back end components
	* Web APIs are usually accessed over HTTP and are usually handled via Web Servers
* **SOAP**:
	* Simple Objects Access
	* shares data through XML, where request is made in XML through HTTP request
	* response also XML
	* useful for transferring structured data (i.e. entire class object) or even binary data
	* often used with serialized objects
	* may be difficult to use for beginners or requires long/complicated queries for smaller queries
* **REST**:
	* Representational Exchange State Transfer
	* shares data through URL Path
	* usually returns data in JSON format
	* REST APIs usually break web app functionality into smaller APIs
	* CRUD Semantics -> GET, POST, PUT and DELETE

## Back-end vulnerabilities
### Common Web Vulnerabilities
* **Broken Access Authentication/Access Control**
	* among the most common and dangerous vulnerabilities
	* Broken Authentication refers to the ability of an attacker to bypass authentication functions
	* Broken Access Control refers to the ability of an attacker to access pages/features to which he should have no access to
* **Malicious File Upload**
	* upload malicious scripts
	* not properly validating the uploaded files 
	* may allow to execute commands on the server
	* basic vulnerability, but developers often not aware or bad implementation of file checks
* **Command Injection**
	* web apps often execute local OS commands
	* if not properly filtered/sanitized -> attacker is able to inject his own commands to be executed on the server
* **SQLInjection**:
	* also occurs when input is not properly sanitized/validated
	* attacker is able to inject his own SQL commands, retrieving data or just causing damage to the database

### Public Vulnerabilities
* most critical back end vulnerabilites are those that can be attacked externally and leveraged to take control over the back-end server
* usually caused by coding mistakes
* **Public CVE**:
	* Common Vulnerabilities and Exposures
	* vulnerabilities shared publicly and scored
	* once we identify version of web app -> search for CVE on online databases like ExploitDB, Rapid7 DB or Vulnerability Lab
* **Common Vulnerability Scoring System (CVSS)**
	* open-source industry standard for assesing the serverity of security vulnerabilities
	* formula using Base, Temporal and Environmental
		* Base metrics produce a score from 0 to 10 (modifed by the other two if given)
		* Severity:
			* None: 0.0
			* Low: 0.1 - 3.9
			* Medium: 4.0 - 6.9
			* High: 7.0 - 8.9
			* Critical: 9.0 - 10.0
* Back-end server vulnerabilities
	* the most critical vulnerabilities are found in web servers as they are publicly accessible via TCP
	* vulns in back-end server or database -> usually utilized after gaining access to the back-end server/network