
### Checklist 

- View SSL certificates for usernames
- View Source code
- Nikto
- Burp
- Check /robots.txt, .htaccess, .htpasswd, sitemap.xml
- Check HTTP Request
- Run Burp Spider
- View Console
- Check OPTIONS
- HTTP PUT / POST File upload
- Parameter fuzzing with wfuzz
- Browser response vs Burp response
- Shell shock (cgi-bin/status)
- Cewl wordlist and directory bruteforce
- Apache version exploit & other base server exploits

### Enum

```
// Fingerprinting
nmap -p80 --script=http-enum 10.10.10.10

// Dir Bust
gobuster dir -u 10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -t 5

// Nikto
nikto -h 10.10.10.10 -p 80,443

// Search APIs
gobuster dir -u http://10.10.10.10:8000 -w /usr/share/wordlists/dirb/common.txt -p /home/kali/pattern    

//Curl 
curl -i http://192.168.246.16:5002/books/v1  
```


### XSS
```
Special Characters
	< > ' " { } ;
```