# Metasploitable 2

## Enumeration
Discover the machine using `netdiscover -r <Network>/24`
### Nmap 
```
PORT      STATE SERVICE
21/tcp    open  ftp      vsftpd 2.3.4
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
3632/tcp  open  distccd
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
6697/tcp  open  ircs-u
8009/tcp  open  ajp13
8180/tcp  open  unknown
8787/tcp  open  msgsrvr
36260/tcp open  unknown
37163/tcp open  unknown
50493/tcp open  unknown
59089/tcp open  unknown
```

TCP ports 512, 513, and 514 are collectively referred to as the "r" services because they support a set of remote commands in Unix/Linux systems that start with "r" 
(like rlogin, rsh, and rexec). These services were historically used for remote login and execution but are now considered insecure and outdated due to their lack of encryption.

It can be log in with `rlogin -l root <ip address>`, So first thing to harden this machine is to disable these ports.

Since port 111 rpcbind is exposed, we can use `rpcinfo` to enumerate all rpc-based services. Lets do `rpcinfo -p <IP>` 
```
   program vers proto   port  service
    100000    2   tcp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  35413  status
    100024    1   tcp  37163  status
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100021    1   udp  37525  nlockmgr
    100021    3   udp  37525  nlockmgr
    100021    4   udp  37525  nlockmgr
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100021    1   tcp  50493  nlockmgr
    100021    3   tcp  50493  nlockmgr
    100021    4   tcp  50493  nlockmgr
    100005    1   udp  58135  mountd
    100005    1   tcp  59089  mountd
    100005    2   udp  58135  mountd
    100005    2   tcp  59089  mountd
    100005    3   udp  58135  mountd
    100005    3   tcp  59089  mountd
```
Let's run `showmount -e 192.168.65.133` to see what can be exported through nfs. `/*` comes back out, so it means everyone can mount root to there computer and use the machine.

### Port 21
Double check what's running on the port & version using `telnet <IP> 21`

`search vsftp` in `vfsconfig`

Set options and Run the exploit. Now we can `cat /etc/shadow` and crack the hashes offline

### Telnet (Capture telnet traffic and catch people in 4k)

Lets attack Telnet and show that data is all sent in clear text. 

First lets turn on wireshark and start a capture. If we then run `telnet <IP>` it comes back with a log in, and after we log in, we can stop the capture and click on any telnet packet. 
Go to follow stream

![telnet](Telnet.png)

Now, lets try to use SSH instead, type `SSH msfadmin@192.168.65.133` and log in with the same password. Below is a what wireshark can see

![SSH](SSH.png)

### Samba 

A service that is for remote compilation of code. like rendering farm or remote computation. It is designed for C and C++, therefore our payload has to be in C or C++.
Lets do a port scan with nmap `nmap -sV -p 3632 192.168.65.133`

```
PORT     STATE SERVICE VERSION
3632/tcp open  distccd distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
```

Lets see if metasploit got a module for it. `msfconsole` and `search distccd`

### SAMBA
