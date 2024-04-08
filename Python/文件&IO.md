- [基本使用](#基本使用)
  - [打开文件的模式](#打开文件的模式)
  - [输出到文件](#输出到文件)
  - [读写字节数据](#读写字节数据)
  - [读写压缩文件](#读写压缩文件)
  - [读取二进制数据到可变缓冲区中](#读取二进制数据到可变缓冲区中)
  - [内存映射的二进制文件](#内存映射的二进制文件)
  - [将文件描述符包装成文件对象](#将文件描述符包装成文件对象)

# 基本使用

## 打开文件的模式

只读模式（'r'）
只写模式（'w'）
追加模式（'a'）
读写模式（'r+'）
二进制只读模式（'rb'）
二进制只写模式（'wb'）
二进制追加模式（'ab'）
二进制读写模式（'r+b' 或 'rb+'）
文本模式读写（'w+'）
二进制读写模式（'w+b'）

## 输出到文件

```python
   with open('output.txt', 'w') as f:
       print('Hello, World!', file=f)

   with open('output.txt', 'w') as f:
       f.write('Hello, World!')

   with open('output.txt', 'w') as f:
       f.writelines(['Hello, ', 'World!'])

>>> import os
>>> if not os.path.exists('somefile'):
...     with open('somefile', 'wt') as f:
...         f.write('Hello\n')
... else:
...     print('File already exists!')
...
File already exists!
>>>
```

## 读写字节数据

```python
# Read the entire file as a single byte string
with open('somefile.bin', 'rb') as f:
    data = f.read()

# Write binary data to a file
with open('somefile.bin', 'wb') as f:
    f.write(b'Hello World')
```

在读取二进制数据的时候，字节字符串和文本字符串的语义差异可能会导致一个潜在的陷阱。 特别需要注意的是，索引和迭代动作返回的是字节的值而不是字节字符串。

```python

with open('somefile.bin', 'wb') as f:
    text = 'Hello World'
    f.write(text.encode('utf-8')) # Hello World

with open('somefile.bin', 'rb') as f:
    data = f.read(16)
    text = data.decode('utf-8')
    print(text)

with open('somefile.bin', 'wb') as f:
    text = b'Hello World'
    f.write(text)   # Hello World

with open('somefile.bin', 'rb') as f:
    data = f.read(16)
    text = data.decode('utf-8')
    print(text) # Hello World
    print(data) # b'Hello World'

```

## 读写压缩文件

```python
# gzip compression
import gzip
with gzip.open('somefile.gz', 'rt') as f:
    text = f.read()

# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'rt') as f:
    text = f.read()

# gzip compression
import gzip
with gzip.open('somefile.gz', 'wt') as f:
    f.write(text)

# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'wt') as f:
    f.write(text)
```

## 读取二进制数据到可变缓冲区中

```python
import os.path

def read_into_buffer(filename):
    buf = bytearray(os.path.getsize(filename))
    with open(filename, 'rb') as f:
        f.readinto(buf)
    return buf
```

## 内存映射的二进制文件

```python
import os
import mmap

def memory_map(filename, access=mmap.ACCESS_WRITE):
    size = os.path.getsize(filename)
    fd = os.open(filename, os.O_RDWR)
    return mmap.mmap(fd, size, access=access)
```

## 将文件描述符包装成文件对象

```python
# Open a low-level file descriptor
import os
fd = os.open('somefile.txt', os.O_WRONLY | os.O_CREAT)

# Turn into a proper file
f = open(fd, 'wt')
f.write('hello world\n')
f.close()

from socket import socket, AF_INET, SOCK_STREAM

def echo_client(client_sock, addr):
    print('Got connection from', addr)

    # Make text-mode file wrappers for socket reading/writing
    client_in = open(client_sock.fileno(), 'rt', encoding='latin-1',
                closefd=False)

    client_out = open(client_sock.fileno(), 'wt', encoding='latin-1',
                closefd=False)

    # Echo lines back to the client using file I/O
    for line in client_in:
        client_out.write(line)
        client_out.flush()

    client_sock.close()

def echo_server(address):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(1)
    while True:
        client, addr = sock.accept()
        echo_client(client, addr)
```