# Kali Linux — Complete Command Cheat Sheet

> A thorough, no-holds-barred cheat sheet focused especially on **archives & extraction** (you said "ALLLLL" — so here's everything I could think of). Use this as a quick reference while working in Kali / Linux.

---

## 🔎 Quick utilities

```bash
file filename                 # Identify file type (very useful before extracting)
ls -la                        # List files
pwd                           # Current directory
cd /path                       # Change directory
mkdir -p dir                   # Make directory (create parents)
rm file                        # Remove file
rm -rf dir                     # Remove dir recursively (be careful)
```

---

## 📦 General archive tips

* **Always run `file` first**: `file archive.ext` helps pick the right extractor.
* Use `-C /path` to extract into a specific directory.
* Use `--strip-components=N` with `tar` to remove leading path components.
* Many modern `tar` implementations auto-detect compression.

---

## 🧰 Universal / Bundled tools

These can often auto-detect and extract many formats:

```bash
# bsdtar (from libarchive) - powerful and auto-detects many formats
bsdtar -xvf archive.ext -C dest/

# GNU tar (modern) often auto-detects compression
tar -xvf archive.tar         # plain tar
tar -xvzf archive.tar.gz     # gzip
tar -xvjf archive.tar.bz2    # bzip2
tar -xvJf archive.tar.xz     # xz

# 7-Zip (p7zip) - very versatile
7z x archive.ext -o./dest/
```

---

## 🗂️ Common archive formats — how to extract (examples)

### `.tar`

```bash
tar -xvf archive.tar                 # extract
```

### `.tar.gz` / `.tgz` (gzip)

```bash
tar -xvzf archive.tar.gz              # extract
# or
gzip -d file.gz                        # produce file; then tar -xvf
```

### `.tar.bz2` (bzip2)

```bash
tar -xvjf archive.tar.bz2
bunzip2 file.bz2                        # decompress single .bz2 file
```

### `.tar.xz` / `.txz` (xz)

```bash
tar -xvJf archive.tar.xz
xz -d file.xz                           # decompress single .xz file
```

### `.zip`

```bash
unzip archive.zip                       # extract into cwd
unzip -l archive.zip                    # list
unzip archive.zip -d /path/to/dest     # extract to directory
```

### `.rar`

```bash
# install unrar or rar-free
unrar x archive.rar                     # extract files keeping paths
unrar e archive.rar                     # extract without paths
```

### `.7z`

```bash
7z x archive.7z                         # extract (keeps paths)
7z l archive.7z                         # list contents
```

### `.gz` (single-file compression)

```bash
gunzip file.gz                          # decompress to file
# or
zcat file.gz | tar xvf -               # if it's tar.gz, or
```

### `.bz2` (single-file)

```bash
bunzip2 file.bz2
```

### `.xz` (single-file)

```bash
unxz file.xz
```

### `.lzma` / `.lz` / `.lz4` / `.lzo`

```bash
# lzma
lzma -d file.lzma
# lzip
lzip -d file.lz
# lz4
lz4 -d file.lz4 file
# lzop
lzop -d file.lzo
```

### `.zst` (zstd)

```bash
unzstd file.zst                        # or: zstd -d file.zst
# to extract a tar.zst
unzstd -c archive.tar.zst | tar xvf -
```

### `.Z` (compress)

```bash
uncompress file.Z
```

### `.tar.lz` / `.tar.lzma` etc.

```bash
# use tar with appropriate flags or pipe through decompressor
xz -dc archive.tar.xz | tar xvf -
lzma -dc archive.tar.lzma | tar xvf -
```

### `.iso` (optical disc image)

```bash
# mount read-only
sudo mount -o loop file.iso /mnt/iso
# extract with 7z
7z x file.iso -o./iso_contents
# or use isoinfo / xorriso for advanced
isoinfo -i file.iso -l
```

### `.img` (disk image)

```bash
# mount raw image (offset may be needed for partitioned images)
sudo mount -o loop,ro file.img /mnt
# use losetup + kpartx for partitioned images
sudo losetup -fP file.img
# find loop device via losetup -a then mount /dev/loopNp1
```

### `.squashfs` (compressed FS often used in live ISOs)

```bash
sudo mount -t squashfs -o loop file.squashfs /mnt/squash
# or
unsquashfs -d dest/ file.squashfs
```

### `.deb` (Debian package)

```bash
# extract contents
ar x package.deb                       # produces control.tar.gz, data.tar.*
# then extract data archive
tar -xvf data.tar.xz                    # or data.tar.gz / data.tar.bz2
# or use dpkg-deb
dpkg-deb -x package.deb dest/
```

### `.rpm` (Red Hat package)

```bash
# convert or extract
rpm2cpio package.rpm | cpio -idmv
# or on RPM-based systems
rpm -ivh --nodeps package.rpm  # installs
```

### `.cpio`

```bash
cpio -idmv < archive.cpio
# or create: find . | cpio -o > archive.cpio
```

### `.cab` (Windows cabinet)

```bash
cabextract file.cab
```

### `.msi`

```bash
# Use less commonly: msiextract or cabextract on internal cabinets
msiextract file.msi -C dest/
```

### `.ar` (Unix archive — used in .deb)

```bash
ar t libfoo.a    # list
ar x libfoo.a    # extract
```

### Windows compressed formats (`.exe` self-extractors, installers)

```bash
# try 7z, unzip, cabextract
7z x installer.exe -o./extracted/
```

---

## 🛠️ Creating archives (examples)

```bash
# tar.gz
tar -czvf archive.tar.gz folder/

# zip
zip -r archive.zip folder/

# 7z
7z a archive.7z folder/

# deb (package building is more involved; simple repackaging):
dpkg-deb -b folder/ package.deb
```

---

## 🔐 Archive options & tricks

* Extract to a directory: `tar -xvf file.tar -C /path/to/dir` or `7z x file -o/path`.
* Strip top directories: `tar --strip-components=1 -xvf archive.tar.gz -C dest/`.
* List contents without extracting: `tar -tvf`, `unzip -l`, `7z l`, `ar t`, `rpm -qpl package.rpm` (if rpm).
* Test archive integrity: `gzip -t file.gz`, `7z t archive.7z`, `zip -T archive.zip`.
* When in doubt use `bsdtar -xvf` or `7z x` — they auto-detect many formats.

---

## 🧩 Repairing & dealing with corrupted archives

* `zip -FF broken.zip --out fixed.zip` (attempt recovery).
* `7z x -r archive.zip` sometimes better at extracting salvageable files.
* For multipart RAR (`.part1.rar`, `.part2.rar`), extract from part1: `unrar x part1.rar`.

---

## 🧪 Example workflows

**Extract into a new directory and strip top folder:**

```bash
mkdir /tmp/extract_here
tar --strip-components=1 -xvzf project-1.2.3.tar.gz -C /tmp/extract_here
```

**Extract unknown archive type safely into a directory:**

```bash
mkdir /tmp/archive_test
file archive.??
bsdtar -xvf archive.?? -C /tmp/archive_test
# inspect contents and then move needed files
```

**Extract Debian package contents for inspection:**

```bash
mkdir /tmp/deb_contents
ar x package.deb
# you'll get control.tar.gz and data.tar.*
tar -xvf data.tar.xz -C /tmp/deb_contents
```

---

## 📥 Installing useful extraction tools (APT)

```bash
sudo apt update
sudo apt install -y p7zip-full p7zip-rar unrar-free unrar bsdtar xz-utils lzma lzip lzop lz4 zstd cabextract rpm2cpio cpio mtools dosfstools sleuthkit
```

---

## ⚙️ Other Kali / Linux cheat sections (brief)

* **Networking:** `ip a`, `ifconfig`, `ss -tulnp`, `nmap -sV target`, `ping`, `traceroute`.
* **Process & system:** `ps aux`, `top`, `htop`, `systemctl status service`.
* **Disk & partitions:** `fdisk -l`, `lsblk`, `mount`, `umount`, `df -h`.
* **Users & perms:** `adduser`, `usermod -aG`, `chmod`, `chown`.
* **Compression utilities:** `gzip`, `bzip2`, `xz`, `zip`, `7z`, `zstd`.

---

## 🧾 Quick reference table (extract) — format → command

```
.tar           -> tar -xvf file.tar
.tar.gz/.tgz   -> tar -xvzf file.tar.gz
.tar.bz2       -> tar -xvjf file.tar.bz2
.tar.xz/.txz   -> tar -xvJf file.tar.xz
.zip           -> unzip file.zip
.rar           -> unrar x file.rar
.7z            -> 7z x file.7z
.gz            -> gunzip file.gz
.bz2           -> bunzip2 file.bz2
.xz            -> unxz file.xz
.zst           -> unzstd file.zst
.iso           -> mount -o loop file.iso /mnt/iso
.deb           -> ar x file.deb && tar -xvf data.tar.*
.rpm           -> rpm2cpio file.rpm | cpio -idmv
.cpio          -> cpio -idmv < file.cpio
.cab           -> cabextract file.cab
.squashfs      -> unsquashfs file.squashfs OR mount -t squashfs -o loop
```

---

## ✅ Final tips

* Keep extraction tools installed (`p7zip`, `bsdtar`, `unrar`, `xz-utils`, `zstd`).
* Use `file` and `bsdtar`/`7z` when unsure.
* Practice safe extraction: inspect contents (`list`) before running any scripts found in archives.

---

*Prepared for you — exhaustive extraction commands included. Want this exported as a downloadable PDF or a printable one-page cheat sheet?*
