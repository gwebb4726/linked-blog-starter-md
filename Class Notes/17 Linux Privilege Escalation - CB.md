
## 17.1 Enumerating Unix

Manual 

```
cat /etc/issue
cat /etc/os-release
uname -a

ps aux

ip a
routel
ss -anp
cat /etc/iptables/rules.v4

ls -lah /etc/cron*
crontab -l
sudo crontab -l

dpkg -l

find / -writable -type d 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

cat /etc/fstab
mount
lsblk

lsmod
/sbin/modinfo <kernel_mod>
```

Automated

```
KALI:/usr/bin/unix-privesc-check
TARGET: ./unix-privesc-check standard > output.txt

linpeas: https://github.com/topics/linpeas

lin enum: https://github.com/rebootuser/LinEnum
```


## 17.2 Exposed Confidential Information