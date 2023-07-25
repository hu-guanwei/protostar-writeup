manipulate invisible characters using Python2

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
