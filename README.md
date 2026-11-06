# Wi-Fi Packet Capture & TLS Inspection with Wireshark

A practical network traffic analysis project inspecting live Wi-Fi packets on Windows using Wireshark. The project covers Domain Name System (DNS) resolution, analyzing Transport Layer Security (TLS) handshake parameters for `networkwalks.com`, and troubleshooting packet capture challenges.

---

## Project Objectives

* Understand how the operating system and web browser resolve domain names via DNS over UDP port 53.
* Inspect the TLS cryptographic handshake to identify Server Name Indication (SNI), transport layer port numbers, and negotiated TLS versions.
* Diagnose and resolve common packet capture issues, including DNS caching, administrative elevation restrictions, and IPv6 vs. IPv4 adapter settings.

---

## Step-by-Step Execution Workflow

1. **Clear System DNS Cache:** Opened Windows Command Prompt as Administrator and ran `ipconfig /flushdns` to purge cached DNS entries so every query is forced onto the network.
2. **Launch Wireshark on Wi-Fi:** Started Wireshark and selected the active **Wi-Fi** adapter showing real-time traffic spikes.
3. **Capture Traffic:** Began capture, navigated to `https://networkwalks.com` in the browser, and stopped the capture once the page loaded.
4. **Filter and Analyze:** Applied targeted display filters to isolate the DNS exchange and TLS handshake frames.

---

## Task Answers & Evidence Breakdown

| # | Question | Answer | Wireshark Frame / Field |
|---|---|---|---|
| **1** | What is the IP address of your own laptop? | **`192.168.137.160`** | Client Source IP (`ip.src`) in DNS queries & TLS Client Hello |
| **2** | What is the IP address of your DNS server? | **`192.168.137.1`** | Gateway/DNS IP (`ip.dst` in query, `ip.src` in response) |
| **3** | What is the IP address of `networkwalks.com`? | **`192.232.216.135`** | Answer section A-record in Packet #1231 |
| **4** | Which website name (Server Name) is sent in the TLS Client Hello? | **`networkwalks.com`** | SNI extension (`server_name`) in Packet #1235 |
| **5** | Which server port is used for the secure connection? | **`443`** | TCP Destination Port (`tcp.dstport`) in Packet #1235 |
| **6** | Which TLS version does the website use? | **`TLSv1.3`** | Negotiated protocol in Server Hello Packet #1238 |

---

### Task 1 & 2: Laptop IP & DNS Server IP

* **Wireshark Filter:** `dns.qry.name == "networkwalks.com"`
* **Finding:** In packet #1230, the laptop sends a DNS standard query from source IP **`192.168.137.160`** to local DNS resolver IP **`192.168.137.1`**. Packet #1231 shows the response originating from `192.168.137.1` back to the laptop.
<img width="1365" height="721" alt="Screenshot 2026-11-06 211205(2)" src="https://github.com/user-attachments/assets/6f9eb432-3101-4fe9-b274-73b4cda4038d" />

![Laptop and DNS Server IP](Screenshot%202026-11-06%20211205(2).png)

---

### Task 3: Website IP Address for networkwalks.com

* **Wireshark Filter:** `dns.qry.name == "networkwalks.com"`
* **Finding:** Packet #1231 contains the DNS Answer Resource Record (`Answer RRs: 1`), confirming that `networkwalks.com` maps to IPv4 address **`192.232.216.135`**.

<img width="1365" height="721" alt="Screenshot 2026-11-06 211205(1)" src="https://github.com/user-attachments/assets/41514014-6eb9-41d8-9d01-2e95fb996f45" />


---

### Task 4: Server Name in TLS Client Hello

* **Wireshark Filter:** `tls.handshake.type == 1`
* **Finding:** Packet #1235 represents the `Client Hello` sent from the laptop to initiate the secure session. Under the `server_name` extension (Server Name Indication / SNI), the hostname requested is **`networkwalks.com`**.

<img width="1365" height="717" alt="Screenshot 2026-11-06 214255(1)" src="https://github.com/user-attachments/assets/601608fb-9378-4e51-9246-f564411438af" />


---

### Task 5: Secure Connection Server Port

* **Wireshark Filter:** `tls.handshake.type == 1`
* **Finding:** Expanding the Transmission Control Protocol (TCP) header in packet #1235 shows **Destination Port: `443`**, the standard port dedicated to HTTPS/TLS traffic.

<img width="1365" height="714" alt="Screenshot 2026-11-06 214351(1)" src="https://github.com/user-attachments/assets/2738e846-56ab-42d0-8082-5e0608049c1d" />

---

### Task 6: Website TLS Version

* **Wireshark Filter:** `tls.handshake.type == 2`
* **Finding:** Packet #1238 contains the `Server Hello` message from `192.232.216.135`. The server confirms cryptographic parameters and establishes the session using **`TLSv1.3`**.

<img width="1365" height="353" alt="Screenshot 2026-11-06 215452(1)" src="https://github.com/user-attachments/assets/b130b5bf-091b-49c8-b330-8e3f12ef5473" />

---

## Challenges Faced & Troubleshooting Solutions

### 1. "The requested operation requires elevation"
* **Problem:** Executing `ipconfig /flushdns` in a standard command prompt failed with an access error.
* **Solution:** Relaunched Command Prompt with administrative rights via **Run as Administrator**, which allowed system-level clearing of the resolver cache.

### 2. Wireshark Captured No DNS Packets
* **Problem:** After opening `networkwalks.com`, searching `dns.qry.name` produced zero packets.
* **Cause:** Modern browsers cache DNS entries internally and default to **DNS over HTTPS (DoH)**, sending lookups as encrypted HTTPS data on port 443 instead of visible UDP port 53 packets.
* **Solution:**
  * Cleared internal browser host caches (`chrome://net-internals/#dns`).
  * Temporarily switched off **Secure DNS (DoH)** in browser security settings.
  * Confirmed packet capture was listening on the active Wi-Fi interface.

### 3. DNS Queried via IPv6 Instead of IPv4
* **Problem:** Initial capture logs resolved queries over IPv6 link-local addresses (`fe80::...`) rather than IPv4.
* This is how the IPv6 ip adress look like
* <img width="1255" height="594" alt="Screenshot 2026-11-06 203004" src="https://github.com/user-attachments/assets/cc536f0e-0b1d-43ad-bf49-9e4ebc3c72a1" />

* **Solution:** Opened Network Connections (`ncpa.cpl`), unchecked **Internet Protocol Version 6 (TCP/IPv6)** on the Wi-Fi adapter properties, and flushed the DNS cache again to route all lookups strictly through IPv4 (`192.168.137.x`).by following thefollowing steps Press Win + R, type ncpa.cpl, and press Enter to open Network Connections.

Right-click my active connection (Wi-Fi) and select Properties.

In the list under "This connection uses the following items", i goes to  Internet Protocol Version 6 (TCP/IPv6).

Uncheck the box next to it.

Click OK to save.

Then i Open Command Prompt as Administrator and flush the cache again.

AUTHOR
Idrrisu Sualihu

