# DISKO-1 — Writeup

**Author:** Darkraicg492
**Writeup by:** Rasheed Gavin (or your name)
**Challenge:** DISKO 1
**Difficulty:** (fill in)
**Category:** Forensics / Disk image

---

## Description

Can you find the flag in this disk image? The challenge provides a disk image file `disko-1.dd.gz` which contains the target data.

---

## Files

* `disko-1.dd.gz` — compressed disk image provided by the challenge

---

## Solution (steps taken)

1. **Download the disk image**

   Place `disko-1.dd.gz` in your working folder (e.g., `~/Downloads`).

2. **Decompress the image**

```bash
# from ~/Downloads
gunzip disko-1.dd.gz
# result: disko-1.dd
```

3. **Search for printable strings matching picoCTF flag format**

```bash
strings disko-1.dd | grep picoCTF
```

4. **Observed output**

```
picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}
```

That output is the flag.

---

## Flag

`picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}`

---

## Explanation / Analysis

* The disk image likely contained the flag as readable ASCII data (not encrypted or carved into file metadata). Running `strings` over the raw image extracted all printable sequences, and filtering for the typical `picoCTF{}` pattern located the flag immediately.
* This is a common beginner forensic technique: `strings` is fast and effective at finding human-readable data within binary blobs or disk images.
* If `strings` had not returned the flag, next steps would include analyzing partitions and filesystems inside the image (e.g., using `fdisk -l`, `mmls`, `fls`, mounting with loopback, or using data carving tools like `foremost` or `photorec`).

## Lessons Learned

* Always try quick, low-effort tools first (`strings`, `grep`) — they often pay off in CTF-style challenges.
* Keep more advanced forensic tools in your toolbox for images where data is hidden inside filesystems or carved files.


