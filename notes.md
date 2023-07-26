## manipulate invisible characters using Python2

### set shell variable
```
user@protostar:~$ a=$(python -c "print 3301")
user@protostar:~$ echo $a
3301
```

```
user@protostar:~$ a=$(python -c "print b'x' + b'\n\r\n\r'")
user@protostar:~$ python
Python 2.6.6 (r266:84292, Dec 27 2010, 00:02:40) 
[GCC 4.4.5] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.environ['a']
'x\n\r\n\r'
```

### manipulate stdin through pip

```
user@protostar:~$ python -c "print 3301" | cat
3301
```

`sc.py`,
```python
a = raw_input()
print(a)
```

```
user@protostar:~$ python -c "print 3301" | python sc.py 
3301
```

```
user@protostar:~$ python -c "print '\n\r'" | python sc.py 


```


## modify return address to hijack flow control

### 32-bit

#### 1
breakpoint before `ret`, next instruction (`eip`) is `ret`
```
[       ]
[       ]
[ret adr] <- esp
```
smash the stack, now the stack layout is,

```
[&bin/sh]
[&adr   ]
[&system] <- esp
```
where `&adr` is another return address you choose.

#### 2
after `ret`, next instruction is `system` with argument `"/bin/sh"`, and after it, `&adr` is to be executed. 

```
[&bin/sh]
[&adr   ]
```

#### 3
after `system("/bin/sh")`, `&adr` is already popped into `eip`, ...


### 64-bit

want `rip=&system` and `rdi=&"/bin/sh"`,

#### 1
breakpoint before `ret`, `rip=ret`

```
[             ]
[             ]
[   ret adr   ] <- rsp
```
smash the stack, make it into

```
[&system      ]
[&"/bin/sh"   ]
[&pop rdi, ret] <-rsp
```

#### 2
after `ret`, now `rip=pop rdi, ret`

```
[&system     ]
[&"/bin/sh"  ] <- rsp
```
#### 3
after `pop rdi, ret`, now `rdi=&"bin/sh"` and `rip=system`