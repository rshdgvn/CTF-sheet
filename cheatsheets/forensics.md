# 🕵️ Forensics CTF Cheatsheet

A full reference for **tools, commands, and techniques** used in CTF Forensics challenge

---

## ⚡ Quick Tips

- Always **make a copy** of the challenge file before modifying it.  
- Check **file type & metadata first** (`file`, `exiftool`).  
- Run **strings + binwalk** early, they reveal a lot quickly.  
- Try common **encodings** (Base64, hex, binary, ROT13).  
- Look for **hidden/embedded files** (append data, ZIP inside image).  
- Use **CyberChef** for weird encodings, magic bytes, or conversions.  
- In PCAPs, start with **Follow TCP Stream** (Wireshark) or `tcpflow`.  
- In memory dumps, dump processes & search for “flag”.  
- Flags are often in **metadata, LSB stego, or leftover space**.  
- Automate repetitive checks with small scripts.  

---

# 🔍 File Analysis

Basic commands to identify, analyze, and extract information from files.

```bash
file suspicious.bin              # Detect file type
strings suspicious.bin | less    # Extract ASCII strings
xxd suspicious.bin | less        # Hex dump
hexdump -C suspicious.bin | less # Alternative hex view
binwalk suspicious.bin           # Scan for embedded files
binwalk -e suspicious.bin        # Extract embedded files
foremost suspicious.bin          # Carve out files
scalpel suspicious.bin           # File carving (configurable)
exiftool image.jpg               # Extract EXIF metadata
exiv2 image.jpg                  # Another metadata parser
```

---

# 🎨 Steganography

Hidden data in images, audio, or video.

```bash
stegseek hidden.jpg wordlist.txt   # Crack steg password
steghide extract -sf image.jpg     # Extract data
zsteg image.png                    # PNG LSB stego
strings -n 6 image.png             # Hidden text
stegsolve                          # GUI steg analysis (images)
stegano-lsb reveal input.png       # Extract LSB data
audacity                           # Inspect audio spectrograms
```

---

#  💾 Disk & Filesystem Forensics

```bash
mmls disk.img                      # Partition layout
fls disk.img                       # List files
icat disk.img 5:10 > output.txt    # Extract inode file
tsk_recover disk.img ./output      # Recover deleted files
autopsy                            # GUI forensic suite
mount -o loop,ro disk.img /mnt     # Mount disk image
bulk_extractor disk.img -o output  # Slack space & carving:
```

---

# 🧠 Memory Forensics

Using Volatility3:

```bash
vol -f memdump.raw windows.pslist        # Running processes
vol -f memdump.raw windows.netscan       # Network connections
vol -f memdump.raw windows.hashdump      # Dump hashes
vol -f memdump.raw windows.cmdline       # Commands executed
vol -f memdump.raw linux.psaux           # Linux processes
strings memdump.raw | grep -i "flag"     # Quick Search

```

---

# 🌐 Network / PCAP Analysis

Wireshark/Tshark

```bash
# Read packets
tshark -r capture.pcap                    
tshark -r capture.pcap -T fields -e ip.src -e ip.dst
tcpdump -r capture.pcap

# Extracting files
wireshark → File → Export Objects → HTTP  
tcpflow -r capture.pcap                   # Extract TCP streams
foremost capture.pcap                     # Carve files

# Filtering
tshark -r cap.pcap -Y "http" -T fields -e http.host -e http.request.uri
tshark -r cap.pcap -T fields -e dns.qry.name
```
---

# 🗂️ Logs & Text

```bash
grep -i "flag" logfile.log
grep -Eo '[A-Za-z0-9]{32,}' logfile.log  # Possible hashes

echo "SGVsbG8=" | base64 -d
echo "68656c6c6f" | xxd -r -p
```

---

# 🛠️ Useful Misc Tools

- **hashcat / john** → crack hashes and passwords  
  ```bash
  hashcat -m 0 hash.txt wordlist.txt
  john --wordlist=rockyou.txt hash.txt
  ```
- **qpdf** → analyze PDFs.
  ```bash
  qpdf --qdf --object-streams=disable file.pdf out.pdf
  ```
- **oletools** → analyze MS Office macros
  ```bash
  olevba suspicious.doc
  ```
- **Hex Editor** → view/modify binary data
  ```bash
  xxd file.bin | less
  xxd file.bin | head
  bvi file.bin
  ghex file.bin
  ```






