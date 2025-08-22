# ASEAN-Japan 2025: P00R R4ns0mw4r3 — Forensics (200 pts)

**Category:** Forensics  
**Points:** 200  
**Attachment:** [Download Attachment](https://drive.google.com/file/d/1eElx3Ivz0-DLCdhkPWG21XsENcv4sDSO/view)

---

## Challenge Description

Your customer has fallen victim to ransomware and is seeking your assistance to decrypt their valuable files. They’ve provided only a memory dump of the command line and the encrypted documents. After conducting OSINT analysis, it appears they were compromised by script kiddies.

---

## Solving the Challenge

1. **Received ZIP File:**
  ```bash
    $ unzip p00r_r4ns0mw4r3.zip
    Archive: extracting p00r_r4ns0mw4r3.zip
      creating: forensic-04/
      extracting: forensic-04/flag.pdf.wcrypt
      inflating: forensic-04/memdump_cmdline_retrieve.txt

    $ cd forensic-04
    $ ls
    flag.pdf.wcrypt memdump_cmdline_retrieve.txt
  ```

  - Unzipped the provided file, discovering two files.
  - Encountered `flag.pdf.wcrypt`, which was corrupted and encrypted.

2. **Analyzed `memdump_cmdline_retrieve.txt`:**
  ```bash
    $ cat memdump_cmdline_retrieve.txt
    Microsoft Windows [Version 10.0.17763.3887]
    (c) 2018 Microsoft Corporation. All rights reserved.
    C:\Users\Administrator\>cd Downloads
    Cloning into 'm4w3rk-CTC' ...
    C:\Users\Administrator\Downloads git clone https://github.com/4ss3mbl3rV/m4w3rk-CTC.git
    remote: Enumerating objects: 18, done.
    remote: Counting objects: 100% (16/18), done.
    remote: Compressing objects: 100% (15/15), done.
    Receiving objects: 100% (18/18), 7.63 MiB | 7.48 MiB/s, done. Receiving objects: 61% (11/18), 7.61 MiB | 7.60
    MiB/s
    Resolving deltas: 100% (1/1), done.
    C:\Users\Administrator\Downloads>dir
    Volume in drive C has no label.
    Volume Serial Number is AE51-7C8A
    Directory of C:\Users\Administrator\Downloads
    08/31/2023 09:05 AM
    <DIR>
    08/31/2023 09:05 AM
    <DIR>
    08/31/2023 09:03 AM
    561,876 2021_Annual_Report.docx
    08/31/2023 03:08 AM
    48,294 flag.pdf
    08/31/2023 09:05 AM
    <DIR>
    08/31/2023 09:04 AM
    m4w3rk-CTC
    08/31/2023 09:03 AM
    1,062,822 NIST.CSWP.04162018.pdf
    08/31/2023 09:03 AM
    2,083,949 NIST.SP.800-225.pdf
    5 File(s)
    10,883,187 wp_2023_security_awareness_report.pdf
    14,640,128 bytes
    3 Dir(s) 32,755,695,616 bytes free
    C:\Users\Administrator\Downloads move m4w3rk-CTC\dist\word.exe
    1 file(s) moved.
    C:\Users\Administrator\Downloads del /F 4w3rk-CTC
    C:\Users\Administrator\Downloads\m4w3rk-CTC\+, Are you sure (Y/N)? Y
    C:\Users\Administrator\Downloads>word.exe
    C:\Users\Administrator\Downloads dir
    Volume in drive C has no label.
    Volume Serial Number is AE51-7CBA
    Directory of C:\Users\Administrator\Downloads
    08/31/2023 09:06 AM
    <DIR>
    08/31/2023 09:06 AM
    08/31/2023 09:06 AM
    08/31/2023 09:06 AM
    <DIR>
    562,432 2021_Annual_Report.docx.wcrypt
    08/31/2023 09:06 AM
    48,336 flag.pdf.wcrypt
    1,063,856 NIST.CSWP.04162018.pdf.wcrypt
    08/31/2023 09:06 AM
    2,085,984 NIST.SP.800-225.pdf.wcrypt
    08/31/2023 09:05 AM
    8,173,809 word.exe
    08/31/2023 09:06 AM
    8,181,792 word.exe.wcrypt
    08/31/2023 09:03 AM
    10,883,187 wp_2023_security_awareness_report.pdf
    7 File(s)
    30,999,396 bytes
    2 Dir(s) 32,756,469,760 bytes free
    C:\Users\Administrator\Downloads move m4w3rk-CTC\dist\word.exe.
    1 file(s) moved.
  ```

   - Found evidence of a tool named `word.exe` used by the attacker to decrypt `flag.pdf`.

3. **Investigated GitHub Repository:**
   - Discovered a cloned repository.
   - Examined commit history to locate the script used by the attacker.

4. **Reversed the Decryption Script:**
   ```python
   # decrypt.py
   from Crypto.Cipher import AES

   # Hardcoded key and IV from memory dump / attacker script
   key = b'\x0e\x6f\xf2\x0e;U^\xbe\x9c\x7f0\x8a=\x97\xdb\xf6\x8a!\x19\xdb\xbc\xc2\xf1\x96\xe6\xb4-'
   iv = b'\xd8\x2d\xe3\x1d\xa4\x53\x03\xd4\x0a'

   cipher = AES.new(key, AES.MODE_CBC, iv)

   input_file = 'flag.pdf.wcrypt'
   output_file = 'flag_decrypted.pdf'

   with open(input_file, 'rb') as fin, open(output_file, 'wb') as fout:
       while True:
           chunk = fin.read(1024 * AES.block_size)
           if len(chunk) == 0:
               break
           decrypted_chunk = cipher.decrypt(chunk)
           fout.write(decrypted_chunk)

   print("Decryption complete. Output written to", output_file)
   ```

   - Created a Python script (`decrypt.py`) using Sublime Text to reverse the decryption process.

5. **Decrypted `flag.pdf.wcrypt`:**
  ```bash
    $ python3 decrypt.py
  ```

   - Ran `decrypt.py`, successfully obtaining `flag.pdf`.

6. **Opened `flag.pdf`:**
   - Verified the presence of the flag inside the decrypted PDF.


## Flag Captured!  
||flag{29fb28c1181de53309b8580906afedc9}||