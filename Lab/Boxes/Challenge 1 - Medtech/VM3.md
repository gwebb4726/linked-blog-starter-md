
##### Nmap

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-20 18:30 EDT
Nmap scan report for 192.168.198.120
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 84727e4cbbff86aeb0030079a1c5af34 (RSA)
|   256 f131e5753136a259f3121b58b4bbdc0f (ECDSA)
|_  256 5a059cfc2f7b7e0b81a620485a1d827e (ED25519)
80/tcp open  http    WEBrick httpd 1.6.1 (Ruby 2.7.4 (2021-07-07))
|_http-title: PAW! (PWK Awesome Website)
|_http-server-header: WEBrick/1.6.1 (Ruby/2.7.4/2021-07-07)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=7/20%OT=22%CT=1%CU=34062%PV=Y%DS=4%DC=T%G=Y%TM=64B9B5A
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=FE%GCD=1%ISR=108%TI=Z%II=I%TS=A)OPS(O1=M5
OS:51ST11NW7%O2=M551ST11NW7%O3=M551NNT11NW7%O4=M551ST11NW7%O5=M551ST11NW7%O
OS:6=M551ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%D
OS:F=Y%T=40%W=FAF0%O=M551NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0
OS:%Q=)T2(R=N)T3(R=N)T4(R=N)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T
OS:6(R=N)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=85B
OS:6%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   29.19 ms 192.168.45.1
2   29.22 ms 192.168.45.254
3   29.33 ms 192.168.251.1
4   29.89 ms 192.168.198.120

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.62 seconds

```



###### Dir bust

```
gobuster dir -u 192.168.198.210 -w /usr/share/wordlists/dirb/common.txt -t 5

/404                  (Status: 200) [Size: 4328]
/about                (Status: 301) [Size: 44] [--> http://192.168.198.120/about/]
/assets               (Status: 301) [Size: 46] [--> http://192.168.198.120/assets/]
/index                (Status: 200) [Size: 4649]
/index.html           (Status: 200) [Size: 4649]
/robots.txt           (Status: 200) [Size: 36]
/sitemap.xml          (Status: 200) [Size: 503]
/static               (Status: 301) [Size: 46] [--> http://192.168.198.120/static/]

WEBrick/1.6.1 (Ruby/2.7.4/2021-07-07)

```


##### nikto
```
nikto -h 192.168.198.120 -p 80

//nothing much here
```