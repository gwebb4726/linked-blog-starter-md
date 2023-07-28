

## 8.2.- Web Application Assessment 

##### Fingerprinting with nmap 
- http-enum performs initial fingerprinting of a server 

##### Tech Stack Identification with Wappalyzer
- performs passive information gathering 
- sign up for a free account www.wappalyzer.com
	- query desired domain
- Technology stacks generally consist of a host operating system, web server software, database software, and a frontend/backend programming language.

#### Brute Force with Gobuster 
- use to map all publicly accessible files and directories 
- common parameters
	- -u = ip
	- -w = wordlist 
	- -t = running threads 
- worldists in /usr/share/wordlists/dirb/

#### Security Testing with Burp Suite
- Proxy tool
	- intercepts traffic between webserver and kali box
	- turn intercept off for continuous traffic flow
	- need to configure firefox browser to send traffic to 127.0.0.1:8080
- Repeater Tool
	- craft or modify http requests  (proxy -> http history -> send to repeater)
- Intruder Tool
	- put target in /etc/hosts file 
	- can be used to brute force login 
		- attempt login -> http history -> send to intruder
		- select password value and supply simple list as payload
	

## 8.3 - Web Application Enumeration

#### Debugging page content 
- firefox debugger tool
	- reveals web page programming languages 
	- {} at the bottom print code legibly 
- inspector tool
	- highlight input fields to find hidden form fields in HTML source

#### Inspecting HTTP Response Headers and Sitemaps
- searching server responses often reveals name, software, and sometimes version number 
	- use burp proxy or network tool
- sitemaps help web engine bots index sites -> reveals publicly accessible web pages
	- robots.txt reveals sensitive pages

#### Enumerating and Abusing APIs
- API = interface responsible for interacting with back end logic and providing a solid backbone of functions for a web application
- Representational State Transfer (REST) = specific API used for authentication + more purposes 
- API naming convention = `/api_name/v[num]`
- Use Gobuster to brute force API endpoints 
	- give gobuster api `/api_name/v[num]` format with -p 
	- manually test discovered APIs with curl or automate with Burp


## 8.4 Cross Site Scripting

#### Stored vs Reflective XSS
- Stored XSS
	- exploit payload is stored/cached by the server
		- web app will retrieve and display payload to all users visiting the site
	- stored XSS usually exists wherever user content can be stored and retrieved (comments, forums, etc.)
- Reflected XSs
	- payload in a crafter request or link
	- only attacks the user visiting the link
	- often in search fields and results
- Both reflected and stored XSS come in server, client, or DOM based versions
- DOM based XSS
	- Takes place solely in a page's Document Object Model
	- occurs when a browser parses a page's content 
- XSS scripts run under the context of the user/the user's browser

#### JavaScript Refresher
- High level programming language
- Browsers process a servers HTTP response and contained HTML -> building a DOM tree and rendering it 
- DOM consits of all forms, inputs, images etc.
- JavaScript accesses and modifies a page's DOM, providing more interaction
	- Means we can also inject javascript code into an app to access the DOM
- For complex JavaScript, use console to test code

#### Identifying XSS vulnerabilities 
- Search web app for input fields that accept unsanitized input
	- Test input field with special characters to determine if any are unfiltered 
	- Common special characters = `< > ' " { } ;`
- URL and HTML encoding most common for exploitation

#### Basic XSS 
- XSS can be used to steal cookies 
	- two important ones are `Secure` and `HttpOnly`
	- `Secure` instructs browser to only send cookie over encrypted comms
	- `HttpOnly` instructs browser to deny JavaScript access to the cookie
		- if not set, can use XSS payload to steal the cookie



#### CV

```
Nmap
	nmap -p80 --script=http-enum 10.10.10.10'

Gobuster
	gobuster dir -u 10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -t 5
	gobuster dir -u http://10.10.10.10:8000 -w /usr/share/wordlists/dirb/common.txt -p /home/kali/pattern     //API
```


### Exercises

8.3.2
```
##Find APIs
gobuster dir -u http://192.168.246.16:5002 -w /usr/share/wordlists/dirb/common.txt -p /home/kali/pattern 

##Test if APIs responsive
curl -i http://192.168.246.16:5002/books/v1    

##Gobuster vulnerable API
gobuster dir -u http://192.168.246.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt

```

8.3.3
```
##Inspected with FF developer -> nothing crazy

##Dir bust
gobuster dir -u http://192.168.231.52 -w /usr/share/wordlists/dirb/big.txt 

/robots.txt           (Status: 200) [Size: 123]  flap pt1
/sitemap.xml          (Status: 200) [Size: 579]  flag pt2 

```

8.3.4
```
very easy, just look a the URL
```

8.3.5
```
Inspect HTTP Response and base64 decode
```

