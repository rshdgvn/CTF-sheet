# File Types Cheat Sheet — README

> Comprehensive reference of common file types you’ll encounter in CTFs and investigations: what they are, identifying headers/magic bytes, quick detection commands, how to extract or open them, and forensic/CTF tips.

---

## 🎯 Purpose

A single README you can search fast: identifies file formats by magic/header, lists commands to inspect/extract, and gives short notes on what to try during CTF/forensics work.

---

## 🧭 Quick tools (use these first)

```bash
file <file>
xxd -l 64 <file> | sed -n '1,20p'    # view first bytes in hex+ASCII
hexdump -C -n 64 <file>
binwalk <file>
strings <file> | less
```

---

## 🗂️ Common filetypes

Each entry: **Name** — *what*, **Magic/Header (hex / ASCII)**, **Commands / How to handle**, **Tips**.

### 1. ELF (Linux executable)

* What: Linux binary executable / shared object.
* Magic: `0x7f 45 4c 46` → `.ELF`.
* Commands:

  ```bash
  ```

file a.out
readelf -h a.out
objdump -d a.out
strings a.out | less
ldd a.out

````
- Tips: check `readelf`, `objdump`; search for ROP gadgets with `ropper`; look for static strings and GOT/PLT leaks.

---

### 2. PE (Windows Portable Executable: .exe/.dll)
- What: Windows executable or DLL.
- Magic: `4d 5a` → `MZ` (DOS stub), `PE\0\0` later.
- Commands:
  ```bash
file file.exe
objdump -f file.exe   # or use `pefile` python library
strings file.exe | less
````

* Tips: run `pefile`/Detect-It-Easy; inspect resources; check for embedded EXEs or installers.

---

### 3. Mach-O (macOS)

* What: macOS binary / dylib.
* Magic: `cf fa ed fe` / `ce fa ed fe` / `ca fe ba be` (fat)
* Commands:

  ```bash
  ```

file binary
otool -h binary

````
- Tips: similar to ELF but mac-specific tools (otool, nm).

---

### 4. PDF
- What: Portable Document Format.
- Magic: `%PDF-` (ASCII 25 50 44 46 2D)
- Commands:
  ```bash
pdfinfo file.pdf
strings file.pdf | grep -i flag
pdfid.py file.pdf
````

* Tips: check for embedded files, JavaScript, and obj streams (use `pdf-parser.py`, `pdfid`).

---

### 5. DOCX / XLSX / PPTX (Office Open XML)

* What: ZIP-based Office files (modern MS Office)
* Magic: ZIP (`50 4B 03 04`) and internal `word/` or `xl/` folders
* Commands:

  ```bash
  ```

file docx
unzip -l file.docx
unzip file.docx -d docx_contents
strings docx_contents/word/document.xml | less

````
- Tips: look inside `word/document.xml`, `rels`, macros (if present, check `vbaProject.bin`). Use `oletools` for old .doc macros.

---

### 6. DOC (old MS Word, OLE)
- What: Older binary Office documents (OLE compound file)
- Magic: `D0 CF 11 E0 A1 B1 1A E1` (OLE)
- Commands:
  ```bash
file file.doc
oletools/olevba file.doc
````

* Tips: use `oletools` to extract macros; search for obfuscated strings and shell commands.

---

### 7. ZIP

* What: Compressed archive format.
* Magic: `50 4B 03 04` (`PK..`)
* Commands:

  ```bash
  ```

unzip -l file.zip
7z l file.zip
binwalk file.zip

````
- Tips: check for path traversal or nested archives; try `zipinfo`; inspect for hidden files in comments.

---

### 8. TAR / TAR.GZ / TGZ / TAR.BZ2 / TAR.XZ
- What: Tape archive (tar) often compressed with gzip/bzip2/xz.
- Magic: tar has no fixed magic for plain, but compressed layers have gzip `1f 8b`, bzip2 `42 5a 68`, xz `fd 37 7a 58 5a 00`.
- Commands:
  ```bash
tar -tf archive.tar
tar -xvzf archive.tar.gz
tar -xvjf archive.tar.bz2
tar -xvJf archive.tar.xz
````

* Tips: use `--strip-components` to avoid path traversal; watch for nested tarballs.

---

### 9. GZIP (.gz)

* What: single-file gzip compression.
* Magic: `1f 8b`.
* Commands:

  ```bash
  ```

gunzip file.gz
zcat file.gz | less

````
- Tips: often combined with tar (`.tar.gz`).

---

### 10. BZIP2 (.bz2)
- What: bzip2 compressed.
- Magic: `42 5a 68` (`BZh`)
- Commands:
  ```bash
bunzip2 file.bz2
````

---

### 11. XZ (.xz)

* What: xz compressed.
* Magic: `fd 37 7a 58 5a 00`
* Commands:

  ```bash
  ```

unxz file.xz

````

---

### 12. 7z
- What: 7-Zip archive.
- Magic: `37 7A BC AF 27 1C`
- Commands:
  ```bash
7z l file.7z
7z x file.7z
````

* Tips: supports many compressed types; good for nested extraction.

---

### 13. RAR

* What: RAR archive.
* Magic: `52 61 72 21 1A 07 00` (`Rar!..`)
* Commands:

  ```bash
  ```

unrar l file.rar
unrar x file.rar

````

---

### 14. ISO (optical image)
- What: CD/DVD filesystem image.
- Magic: `43 44 30 30 31` sometimes; use `file`.
- Commands:
  ```bash
sudo mount -o loop file.iso /mnt/iso
7z x file.iso -oiso_contents
isoinfo -i file.iso -l
````

* Tips: check for squashfs/embedded filesystems.

---

### 15. IMG (disk image)

* What: raw disk or partition image.
* Magic: varies; partition tables show MBR/GPT signatures.
* Commands:

  ```bash
  ```

fdisk -l image.img
losetup -Pf image.img  # map partitions
kpartx -av image.img
mount -o ro,loop,offset=$((START*512)) image.img /mnt

````
- Tips: calculate partition offsets from `fdisk -l` and mount read-only.

---

### 16. SQUASHFS (.squashfs)
- What: compressed read-only filesystem (live ISOs).
- Magic: `68 73 71 73`? (use file)
- Commands:
  ```bash
unsquashfs file.squashfs -d outdir
mount -o loop file.squashfs /mnt
````

---

### 17. DEB (Debian package)

* What: Debian package (ar archive containing control and data tarballs).
* Magic: `21 3C 61 72 63 68 3E`? Actually begins with `!<arch>` (ar) `21 3C 61 72 63 68 3E` & `debian-binary` inside.
* Commands:

  ```bash
  ```

ar t package.deb
ar x package.deb

# then extract data.tar.*

tar -xvf data.tar.xz

# or dpkg-deb -x package.deb dest/

````
- Tips: inspect `control` for maintainer scripts; check `data` for installed files.

---

### 18. RPM
- What: Red Hat package format.
- Magic: `ED AB EE DB`? (use file)
- Commands:
  ```bash
rpm2cpio package.rpm | cpio -idmv
````

---

### 19. MSI / CAB

* What: Windows installer / cabinet files.
* Magic: MSI is OLE or database; CAB `4D 53 43 46` (`MSCF`)
* Commands:

  ```bash
  ```

cabextract file.cab
msiextract file.msi -C outdir

````

---

### 20. APK (Android package)
- What: ZIP-based Android package containing DEX, resources.
- Magic: ZIP header `50 4B 03 04` and `AndroidManifest.xml` / `classes.dex` inside.
- Commands:
  ```bash
unzip -l app.apk
apktool d app.apk -o app_src      # decode resources
jadx-gui app.apk                  # decompile
````

* Tips: check `classes.dex` (use `dex2jar`, `jadx`) and `AndroidManifest.xml`.

---

### 21. JAR / WAR (Java archives)

* What: ZIP-based Java archive with `.class` files.
* Magic: ZIP `50 4B 03 04`
* Commands:

  ```bash
  ```

jar tf file.jar
jd-gui file.jar

````

---

### 22. SQLITE (.sqlite, .db)
- What: SQLite database file.
- Magic: `53 51 4C 69 74 65 20 66 6F 72 6D 61 74` → `SQLite format 3\0`
- Commands:
  ```bash
sqlite3 file.db ".tables"
sqlite3 file.db "select * from table limit 10;"
````

* Tips: extract strings and blobs; look into `sqlite_master` for schema and hidden tables.

---

### 23. PCAP / PCAPNG (packet capture)

* What: network capture files.
* Magic: PCAP `d4 c3 b2 a1` or `a1 b2 c3 d4` (endianness); PCAPNG `0a 0d 0d 0a`
* Commands:

  ```bash
  ```

file capture.pcap
tshark -r capture.pcap -q -z conv,ip
tcpdump -r capture.pcap -nn
wireshark capture.pcap

````
- Tips: use `tshark` for automation, `tcpflow` or `foremost` to extract files from streams.

---

### 24. PNG (image)
- What: Portable Network Graphics.
- Magic: `89 50 4E 47 0D 0A 1A 0A` → `.PNG....`
- Commands:
  ```bash
file img.png
pngcheck img.png
exiftool img.png
binwalk img.png
zsteg img.png
````

* Tips: check ancillary chunks (tEXt/iTXt), appended data, LSB stego (zsteg), and use `binwalk` to extract embedded files.

---

### 25. JPEG / JPG

* What: JPEG image.
* Magic: `FF D8 FF` (start); `FF D9` end
* Commands:

  ```bash
  ```

file img.jpg
exiftool img.jpg
jpeginfo -c img.jpg
binwalk img.jpg

````
- Tips: check EXIF, comments, and appended data; `strings` may reveal embedded shells or flags.

---

### 26. GIF
- What: GIF image / animation.
- Magic: `47 49 46 38 39 61` (`GIF89a`) or `GIF87a`
- Commands: `file`, `exiftool`, `gifsicle -I`.

---

### 27. BMP
- What: Bitmap image.
- Magic: `42 4D` (`BM`)
- Commands: `file`, `strings`, `exiftool`.

---

### 28. WAV (audio)
- What: RIFF WAV audio file.
- Magic: `52 49 46 46` (`RIFF`) and `57 41 56 45` (`WAVE`) later.
- Commands:
  ```bash
file sound.wav
sox sound.wav -n stat
audacity sound.wav # GUI to inspect spectrogram
````

* Tips: audio stego — inspect spectrograms (Audacity) and `strings`.

---

### 29. MP3

* What: MPEG audio stream.
* Magic: `49 44 33` (ID3 tag) or frame sync `FF FB`.
* Commands: `mp3info`, `ffmpeg -i file.mp3 -f wav - | ...`

---

### 30. MP4 / MKV / AVI (video)

* What: Container formats.
* Magic: MP4 often `00 00 00 .. ftyp`, MKV `1A 45 DF A3` (EBML)
* Commands:

  ```bash
  ```

ffprobe file.mp4
mediainfo file.mkv
ffmpeg -i file.mp4 -map 0:1 out_audio.wav

````
- Tips: check subtitle tracks, attachments, and metadata; extract streams with `ffmpeg`.

---

### 31. TAR.LZMA / LZMA / LZIP / LZ4 etc.
- What: other compression formats (lzma, lz4, lzip)
- Commands: `lzma -d`, `lz4 -d`, `lzip -d` or use `7z`.

---

### 32. GPG / PGP encrypted files
- What: OpenPGP encrypted message/file.
- Magic: ASCII armored `-----BEGIN PGP MESSAGE-----` or binary packets.
- Commands:
  ```bash
gpg --list-packets file.gpg
gpg --decrypt file.gpg  # needs private key/password
````

* Tips: search for ASCII armored blocks in text, check for keys and passphrases in memory or files.

---

### 33. TAR/GZ inside images or other files (carved nested)

* What: archives embedded inside other files (steganography, container files)
* Commands:

  ```bash
  ```

binwalk -e file
foremost -i file -o carved

```
- Tips: always run `binwalk` and `foremost` on unknown blobs.

---

## 🛠️ Generic handling recipes
- **Identify:** `file`, `xxd -l 64`, `binwalk`.
- **List archive contents:** `unzip -l`, `tar -tf`, `7z l`.
- **Extract safely:** `7z x` or use format-specific extractors into a new directory.
- **Search for flags:** `strings`, `grep -a -R 'flag{' .`, `yara` rules.
- **Inspect metadata:** `exiftool`, `pdf-parser.py`, `oletools`.

---

## ✅ Final tips
- Keep a toolkit: `file`, `xxd`, `binwalk`, `foremost`, `7z`, `unzip`, `exiftool`, `sqlite3`, `apktool`, `jadx`, `radare2`, `strings`, `gpg`.
- When in doubt, run `binwalk -Me` and `foremost` on the file—many CTF flags hide in embedded archives or appended data.
- Always work on copies and compute checksums.

---

*Want this exported as a printable PDF, or should I add a downloadable reference table (CSV/JSON) of magic bytes for quick lookup?*

```
