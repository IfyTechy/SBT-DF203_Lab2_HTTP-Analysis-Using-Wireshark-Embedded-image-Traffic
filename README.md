# SBT-DF203 Lab 2: HTTP Analysis Using Wireshark – Embedded Image Traffic

> **Formal Digital Forensics Investigation Report**  
> **Course:** SBT-DF203: Basic Networking Skills for Digital Forensics  
> **Lead Examiner:** Nebeuwa Ifeanyichukwu Raphael  
> **Case Identifier:** `SBT-DF203-Lab2`  
> **Primary Target:** Kali Linux Workstation (`kali@kali`)  
> **Submission Date:** September 10, 2026  

---

## 📌 Executive Summary

This repository contains the investigation artifacts, packet captures, and analytical documentation for **SBT-DF203 Lab 2**. The examination focuses on capturing, analyzing, and reconstructing HTTP traffic carrying an embedded image object using `Wireshark` and `tshark`.

### Key Investigation Highlights
- **Source Evidence Provenance:** Generated a synthetic evidence image (`lab_photo.jpg`) via ImageMagick containing embedded visual provenance markers (Examiner Name & Date) to guarantee authorization and ownership integrity.
- **Traffic Capture & Isolation:** Recorded browser-generated HTTP sessions over the loopback interface (`lo`) on TCP port 80.
- **Multi-Object Verification:** Confirmed that a single HTML page load generates separate, distinct HTTP GET requests for embedded assets (`GET /image.html` → `GET /lab_photo.jpg` → `GET /favicon.ico`).
- **Transport Layer Analysis:** Analyzed TCP stream 0 and proved that the 30,527-byte image payload was carried within a single 30,815-byte TCP segment, identifying buffer thresholds vs. object size dynamics.
- **Forensic Extraction & Integrity Check:** Successfully exported the embedded image from raw PCAP files using `tshark`'s HTTP object-export feature. Cryptographic SHA-256 hashing confirmed **100% bit-for-bit integrity** against the source file.
- **Client Behavior Comparison:** Demonstrated the functional divergence between rendering browsers (automatic DOM parsing & resource fetching) and CLI HTTP clients (`curl`, single explicit request without DOM evaluation).

---

## 🛠️ Environment & Tools Used

| Category | Component / Tool |
| :--- | :--- |
| **Operating System** | Kali Linux (`kali@kali`) |
| **Web Server** | Apache2 (Localhost on Port 80) |
| **Packet Analyzer** | `Wireshark` / `tshark` |
| **CLI & Utilities** | `curl`, `ImageMagick` (`convert`), `file`, `sha256sum` |
| **Capture Interface** | Loopback (`lo`) |

---

## 🔬 Core Investigation Phases

### Part A: Evidence Preparation & Baseline Hashes
Created the HTML page (`image.html`) and synthetic image (`lab_photo.jpg`). Calculated baseline SHA-256 hashes prior to network capture:
- **`lab_photo.jpg` (SHA-256):** `7832609ee980a5035e6e13a862ffa7ba72b15f9753829d0a7cff8693266a6c92`

### Part B: Browser-Generated Traffic Capture
Captured loopback traffic during a private browser session. Saved raw capture to `evidence/image_traffic.pcapng` and generated an identical working copy for analysis.
- **Capture File Hash (SHA-256):** `28c192d0a7d470a313555115e03929a7f4100165385e0fcf21b53bb2001bd519`

### Part C: Multi-Request HTTP Proof
Extracted request/response streams showing browser DOM parsing behavior:
1. `Frame 4`: `GET /image.html` (HTTP 200 - `text/html`)
2. `Frame 8`: `GET /lab_photo.jpg` (HTTP 200 - `image/jpeg`)
3. `Frame 12`: `GET /favicon.ico` (HTTP 404 - Not Found)

*In-Session Caching Note:* Subsequent reloads under Stream 1 only re-requested `image.html`, serving `lab_photo.jpg` directly from the active in-memory session cache.

### Part D: TCP Segmentation & Size Calculations
- **Total TCP Payload (`tcp.len`):** 30,815 bytes
- **HTTP `Content-Length` (Image Body):** 30,527 bytes
- **Calculated HTTP Header Length:** $30,815 - 30,527 = 288 \text{ bytes}$
- *Segmentation Outcome:* Reassembly was not required because the entire response (~30.8 KB) fit below the loopback TCP send-buffer threshold (~32.7 KB).

### Part E: Forensic Object Extraction & Verification
Extracted all HTTP objects using `tshark`:
```bash
tshark -r working/image_traffic_working.pcapng --export-objects http,exported/http_objects
```

**Extracted Image Hash:** `7832609ee980a5035e6e13a862ffa7ba72b15f9753829d0a7cff8693266a6c92`

**Result:** MATCH - Proves lossless, unaltered forensic recovery.

### **Part F:** curl vs. Browser Behavioral Analysis

A separate capture during a curl -v http://127.0.0.1/image.html request confirmed that curl issued only one request (/image.html). Lacking an HTML rendering engine, it did not parse <img src="..."> tags or issue secondary requests for /lab_photo.jpg or /favicon.ico.

### ⚖️ Disclaimer & Limitations

All procedures were executed within an isolated lab environment under explicit authorization for academic digital forensics analysis.



