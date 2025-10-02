# Wireshark for CTFs — README

A compact, practical README that gathers essential Wireshark/TShark tips, filters, workflows, and checklists tailored for Capture The Flag (CTF) and network security challenges. Use this as a quick reference while analyzing PCAPs or capturing live traffic during challenges.

---

## 🎯 Purpose

This README is designed to help you rapidly find flags, detect exfiltration, reconstruct transferred files, and spot covert channels using Wireshark and its command-line sibling, TShark.

---

## 🔁 Capture vs Display filters

* **Capture filters** (BPF, used before capture): e.g. `tcp port 80 and host 10.0.0.5`. Reduces data captured.
* **Display filters** (Wireshark/TShark, protocol-aware): e.g. `http.request.method == "POST"`. Use these to drill into captures.

---

## 🔎 Common display filters (copy/paste)

* `ip.addr == 10.10.14.7`
* `http.request`
* `http.request.method == "POST"`
* `dns.qry.name contains "example"`
* `tls.handshake.type == 1`
* `tcp.analysis.retransmission`
* `frame contains "flag{"`
* `frame matches "flag\\{[A-Za-z0-9_\\-]+\\}"`

---

## 🚀 Fast analysis workflow

1. File → Protocol Hierarchy to see top protocols.
2. Statistics → Endpoints / Conversations to find top talkers.
3. Analyze → Expert Information for anomalies.
4. Search for `flag{`, suspicious base64, or long printable blobs.
5. Follow streams (TCP/UDP/HTTP) and export raw data for further analysis.

---

## 🔧 Reconstructing & extracting data

* Right-click → Follow → TCP Stream (or HTTP/UDP) → Save as Raw/ASCII.
* File → Export Objects → HTTP/SMB/FTP to pull transferred files.
* Use `tshark`/`tcpflow` to automate extraction and reconstruction.

---

## 🔐 TLS decryption tips

* Use `SSLKEYLOGFILE` in the client (browser) and configure Wireshark to use it for TLS decryption.
* RSA private keys only work for non-ephemeral RSA key exchanges (rare with modern TLS/ECDHE).

---

## 🧰 Useful TShark commands

* List HTTP requests:

  ```bash
  ```

tshark -r cap.pcap -Y 'http.request' -T fields -e http.host -e http.request.uri

````
- Extract DNS queries:
  ```bash
tshark -r cap.pcap -Y 'dns.qry.name' -T fields -e dns.qry.name | sort -u
````

* Save HTTP objects:

  ```bash
  ```

tshark -r cap.pcap --export-objects http,./http_objs

```

---

## 🕵️ Heuristics to spot suspicious activity
- Many small DNS queries to randomized subdomains → beaconing/DGA.  
- Long-lived low-traffic TCP sessions → possible reverse shell.  
- Non-standard ports carrying HTTP-like payloads → covert HTTP.  
- Long printable/base64 blobs in frames → potential exfiltration.

---

## 🧾 Quick checklist for CTF PCAP analysis
1. Protocol hierarchy → spot unexpected protocols.  
2. Expert info → look for malformed or suspicious packets.  
3. Endpoints/Conversations → find unusual peers.  
4. Grep frames for `flag{`, `CTF{`, or base64 blobs.  
5. Export objects/follow streams → recover files.  
6. If TLS, try `SSLKEYLOGFILE` or server key (if available).  

---

## 💡 Pro tips
- Turn off name resolution to speed up analysis.  
- Enable TCP reassembly for protocols that use segmentation.  
- Use coloring rules to highlight suspicious traffic types.  
- Automate repetitive tasks with `tshark` scripts.

---

## 📚 Further reading & tools
- Wireshark User Guide (official)  
- TShark documentation  
- tcpflow, binwalk, and other file-reconstruction utilities

---

*This README is optimized for quick lookup during CTFs — want a printable one-page cheat sheet or a sample `tshark` script bundle next?*
```
