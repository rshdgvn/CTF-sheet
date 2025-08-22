# ASEAN-Japan 2025: P00R R4ns0mw4r3 — Forensics (200 pts)

**Category:** Forensics  
**Points:** 200  
**Attachment:** [Download Attachment](https://drive.google.com/file/d/1eElx3Ivz0-DLCdhkPWG21XsENcv4sDSO/view)

---

## Challenge Description

Your customer has fallen victim to ransomware and is seeking your assistance to decrypt their valuable files. They’ve provided only a memory dump of the command line and the encrypted documents. After conducting OSINT analysis, it appears they were compromised by script kiddies.

---

## Solving the Challenge

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
1. **Received ZIP File:**
   - Unzipped the provided file, discovering two files.
   - Encountered `flag.pdf.wcrypt`, which was corrupted and encrypted.

2. **Analyzed `memdump_cmdline_retrieve.txt`:**
   - Found evidence of a tool named `word.exe` used by the attacker to decrypt `flag.pdf`.

3. **Investigated GitHub Repository:**
   - Discovered a cloned repository.
   - Examined commit history to locate the script used by the attacker.

4. **Reversed the Decryption Script:**
   - Created a Python script (`decrypt.py`) using Sublime Text to reverse the decryption process.

5. **Decrypted `flag.pdf.wcrypt`:**
   - Ran `decrypt.py`, successfully obtaining `flag.pdf`.

6. **Opened `flag.pdf`:**
   - Verified the presence of the flag inside the decrypted PDF.

---

✅ **Flag Captured!**
```
