# **Finding Bad Characters**

#just find badchar sending via payload then esp dump thing find the bad chars

#jump to them to find the return value

#Generate a bytearray using mona, and exclude the null byte (\x00) by default. Note the location of the bytearray.bin file that is generated (if the working folder was set per the Mona Configuration section of this guide, then the location should be C:\mona\oscp\bytearray.bin).

#IMP —- use direct bad chars just edit it no neeed to type shit.

#```python
!mona bytearray -b "\x00"     ### use this mona to jmp ###
#```

#Now generate a string of bad chars that is identical to the bytearray. The following python script can be used to generate a string of bad chars from \x01 to \xff:

#```python
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
```

# Update your [exploit.py](http://exploit.py/) script and set the payload variable to the string of bad chars the script generates.

#```python
!mona compare -f C:\mona\oscp\bytearray.bin -a <ESP_address>        ### mona to compare out ###
```

#ESP > follow the dump for more accurate result
