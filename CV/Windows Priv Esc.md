

#### CMD Survey 

```
## System Info + Users ##
systeminfo
whoami
whoami /groups
net user
net localgroup 

## Networking ##
ipconfig /all
route print
netstat -ano

## Installed Software ##
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\"
dir "C:\Program Files\"
dir "C:\Program Files (x86)\"
dir "C:\ProgramData\"

## Processes ##  
tasklist
tasklist /m

```


#### Powershell Survey

```
Get-LocalUser
Get-LocalGroup

Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-Process
```