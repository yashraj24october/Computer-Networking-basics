# Computer Networking Fundamentals
> A complete revision guide covering Networks, OSI Model, HTTP/2, SSL/TLS, and Nginx Configuration.

---

## Table of Contents
1. [What is a Message in Networks?](#1-what-is-a-message-in-networks)
2. [Client-Server Communication](#2-client-server-communication)
3. [Real Example — Opening google.com](#3-real-example--opening-googlecom)
4. [OSI Model — 7 Layers](#4-osi-model--7-layers)
5. [Data Link vs Physical Layer](#5-data-link-vs-physical-layer)
6. [Encapsulation & Decapsulation](#6-encapsulation--decapsulation)
7. [HTTP vs HTTPS](#7-http-vs-https)
8. [HTTP/2 — The Upgrade](#8-http2--the-upgrade)
9. [Encryption — How It Works](#9-encryption--how-it-works)
10. [SSL Certificate & Let's Encrypt](#10-ssl-certificate--lets-encrypt)
11. [Nginx Configuration — Manual OSI](#11-nginx-configuration--manual-osi)

---

## 1. What is a Message in Networks?

A **message** is any unit of data sent from one device to another in a network.

Every message has two parts:

| Part | Description |
|------|-------------|
| **Header** | Control info — source/destination IP, port numbers, checksums, timestamps |
| **Payload** | Actual data — webpage, email body, video chunk, file |

### Key Points:
- Messages are broken into smaller chunks called **packets** (~1500 bytes each)
- Each packet travels **independently** across the network
- Packets are **reassembled** in correct order at the destination

---

## 2. Client-Server Communication

```
Client                    Internet                 Server
(Your PC)  ────────────────────────────────────►  (Web Server)
           ◄────────────────────────────────────
```

| Role | Description |
|------|-------------|
| **Client** | Asks / Requests (your browser) |
| **Server** | Listens and Responds (Google's computer) |

> A website = **a collection of messages** (HTML, CSS, JS, Images) exchanged between browser and server.

---

## 3. Real Example — Opening google.com

```
[0ms]      You press Enter
[1ms]      DNS resolves google.com → IP address
[5ms]      TCP handshake completes
[20ms]     Your request reaches Google's server
[50ms]     Google's response starts arriving
[100ms]    All packets received & reassembled
[101ms]    Browser renders the page!
```

### Step-by-Step:

**Step 1 — DNS Lookup**
```
Your PC  →  DNS Server:  "What is the IP of google.com?"
DNS Server  →  Your PC:  "It's 142.250.80.46"
```

**Step 2 — TCP Handshake**
```
Your PC  →  Google:  SYN     "Hello, I want to connect"
Google   →  Your PC: SYN-ACK "Hello back, I'm ready"
Your PC  →  Google:  ACK     "Great, let's communicate"
```

**Step 3 — HTTP Request**
```
Your PC → Google:
  Header:  From 192.168.1.5:54321 → To 142.250.80.46:80
  Payload: GET / HTTP/1.1 | Host: google.com
```

**Step 4 — HTTP Response**
```
Google → Your PC:
  Header:  Status 200 OK
  Payload: <html>...</html>
  (Split into many packets, reassembled at browser)
```

**Step 5 — Connection Close**
```
Your PC  →  Google:  FIN  "I'm done, goodbye"
Google   →  Your PC: ACK  "Goodbye"
```

---

## 4. OSI Model — 7 Layers

> OSI model is the **journey map of a message** as it travels from your browser to a server.

```
YOUR COMPUTER (Sending)          SERVER (Receiving)

┌─────────────────────┐          ┌─────────────────────┐
│  7. Application     │          │  7. Application     │
│  6. Presentation    │          │  6. Presentation    │
│  5. Session         │          │  5. Session         │
│  4. Transport       │          │  4. Transport       │
│  3. Network         │          │  3. Network         │
│  2. Data Link       │          │  2. Data Link       │
│  1. Physical        │          │  1. Physical        │
└─────────┬───────────┘          └─────────▲───────────┘
          └──────────── Internet ───────────┘

Message goes DOWN your layers → travels → goes UP server's layers
```

### Layer Details:

| Layer | Name | Role | Browser Example |
|-------|------|------|-----------------|
| 7 | Application | User interaction, HTTP requests | Browser creates `GET /` request |
| 6 | Presentation | Encrypt, compress, format data | SSL/TLS encrypts message |
| 5 | Session | Open, manage, close connection | Manages browser-server conversation |
| 4 | Transport | Split into segments, port numbers | TCP splits data, adds port 443 |
| 3 | Network | IP addressing, routing | Adds your IP → Google's IP |
| 2 | Data Link | MAC addressing, error detection | Adds MAC address, checks CRC |
| 1 | Physical | Converts to signals, sends bits | Electrical/optical/radio signals |

### Memory Trick:
```
"All People Seem To Need Data Processing"
  A     P      S     T    N     D      P
  7     6      5     4    3     2      1
```

### Session Layer — Important Note:
```
❌ Wrong: Session layer establishes connection first, then lower layers work
✅ Right:  Lower layers (1,2,3,4) do TCP handshake first
           THEN session layer manages that open connection
```

---

## 5. Data Link vs Physical Layer

> Most confusing layers — but very different jobs!

```
Physical Layer  =  The ROAD       (just carries signals, no thinking)
Data Link Layer =  The DRIVER     (controls, addresses, checks errors)
```

### Physical Layer:
- Converts bits → electrical/optical/radio signals
- Sends raw bits through wire or WiFi
- Does NOT know: where it's going, if data is correct, which device receives it
- **Completely blind and dumb — just sends signals**

### Data Link Layer — 3 Jobs:

**Job 1 — Framing:**
```
┌──────────┬──────────┬──────────────┬──────────┐
│ Dest MAC │ Src MAC  │   DATA       │  CRC     │
│(Router)  │(Your PC) │  (message)   │(checksum)│
└──────────┴──────────┴──────────────┴──────────┘
```

**Job 2 — MAC Addressing:**
- MAC address = physical address burned into every network device
- Only for ONE hop at a time (not full journey)
- Example: Your PC → Router (not Your PC → Google)

**Job 3 — Error Detection (CRC):**
```
Sending:   "My data checksum = 4821"
Receiving: "I recalculate...  4821 ✓ perfect!"
                               4756 ✗ corrupted! resend.
```

### Summary Table:

| | Physical Layer | Data Link Layer |
|--|---------------|-----------------|
| Sends | Raw bits | Structured Frames |
| Uses | Signals | MAC addresses |
| Addressing | None | Next device only (one hop) |
| Error check | None | Yes (CRC checksum) |
| Intelligence | Dumb | Smart |
| Examples | Cable, WiFi, Fiber | Ethernet, Switch |

---

## 6. Encapsulation & Decapsulation

> Message travels like an **assembly line** — each layer adds or removes its part.

### Sending = Encapsulation (wrapping):
```
Layer 7 message
┌─────────────────────────────────────┐
│  Layer 4                            │
│  ┌───────────────────────────────┐  │
│  │  Layer 3                      │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │  Layer 2                │  │  │
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │  Original Data    │  │  │  │
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
Each layer ADDS its header (like gift boxes)
```

### Receiving = Decapsulation (unwrapping):
```
Layer 1 receives signals
→ Layer 2 removes its frame header
→ Layer 3 removes IP header
→ Layer 4 removes port/segment header
→ Layer 7 reads original message ✓
```

---

## 7. HTTP vs HTTPS

| | HTTP | HTTPS |
|--|------|-------|
| Port | 80 | 443 |
| Encryption | None | SSL/TLS |
| Safety | ❌ Data visible | ✅ Data encrypted |
| Browser | 🔓 No padlock | 🔒 Padlock shown |
| Use case | Simple sites | Banking, Login, Shopping |

```
HTTP  → Anyone watching network can READ your data openly
HTTPS → Anyone watching sees only: "xK9#mP2$vL5@nQ8*zR3"
```

---

## 8. HTTP/2 — The Upgrade

### Problem with HTTP/1.1:
```
ONE request at a time = SLOW 🐢
[HTML] → wait → [CSS] → wait → [Image] → wait...
```

### HTTP/2 Solutions:

**1. Multiplexing** — Many requests at once:
```
HTTP/1.1: [HTML]──►wait──►[CSS]──►wait──►[Image]

HTTP/2:   [HTML]  ────────────────────────────►
          [CSS]   ────────────────────────────►
          [Image] ────────────────────────────►
          [JS]    ────────────────────────────►
          All at the same time! 🚀
```

**2. Header Compression (HPACK):**
```
HTTP/1.1: Sends full headers every request (wasteful)
HTTP/2:   Remembers headers, sends only what CHANGED (tiny!)
```

**3. Server Push:**
```
HTTP/1.1: Browser asks for HTML → reads it → asks for CSS (extra trip!)
HTTP/2:   Browser asks for HTML → Server sends HTML + CSS + JS automatically!
```

**4. Binary Protocol:**
```
HTTP/1.1: Text based  → humans can read, computers process slowly
HTTP/2:   Binary      → humans cannot read, computers process FASTER ⚡
```

**5. Stream Priority:**
```
Send HTML first  (priority 1) → most important
Send CSS second  (priority 2) → needed for display
Send images last (priority 3) → less urgent
```

### Performance:
```
HTTP/1.1: Loading 100 files → ~3.5 seconds 🐢
HTTP/2:   Loading 100 files → ~1.2 seconds 🚀
```

> HTTP/2 lives at **Layer 7 (Application Layer)** in OSI model.

---

## 9. Encryption — How It Works

### Asymmetric Encryption (Public/Private Key Pair):

```
🔑 Public Key  = Share with EVERYONE openly (like mailbox slot)
🗝️ Private Key = Keep SECRET on your server only (like mailbox key)

Rule:
What PUBLIC key ENCRYPTS → only PRIVATE key can DECRYPT
What PRIVATE key ENCRYPTS → only PUBLIC key can DECRYPT
```

### Full HTTPS Handshake Flow:

**Phase 1 — Asymmetric (secure key exchange):**
```
Step 1: Server shares certificate (contains Public Key) → Browser
Step 2: Browser verifies certificate with Let's Encrypt ✓
Step 3: Browser generates random Session Key "aBcDeFgH1234"
Step 4: Browser encrypts Session Key with Server's Public Key
Step 5: Encrypted Session Key → Server
Step 6: Server decrypts with Private Key → gets Session Key ✓
```

**Phase 2 — Symmetric (fast data transfer):**
```
Both sides now have: "aBcDeFgH1234"
All data encrypted/decrypted with this same key
Much faster than asymmetric! ⚡
```

### Why Two Types?

| | Asymmetric | Symmetric |
|--|-----------|-----------|
| Security | Very HIGH | HIGH |
| Speed | SLOW | FAST |
| Keys | Public + Private pair | Same key both sides |
| Used for | Exchanging session key | All actual data transfer |

---

## 10. SSL Certificate & Let's Encrypt

### What is a Certificate?
```
Proves: "Are you REALLY yoursite.com or a fake?"

Certificate contains:
┌─────────────────────────────────────────┐
│  Domain:    yoursite.com                │
│  Owner:     Your Name                   │
│  Valid:     90 days                     │
│  Public Key: xK9#mP2$vL5...            │
│  Signed by: Let's Encrypt              │
└─────────────────────────────────────────┘
```

### Browser Certificate Check:
```
Is certificate signed by Let's Encrypt?  ✓
Is Let's Encrypt trustworthy?            ✓  (pre-installed in browser)
Is it for correct domain?                ✓
Is it expired?                           ✓
→ Safe to connect! 🔒
```

### Let's Encrypt + Certbot:
```
Let's Encrypt = FREE certificate authority (issues certificates)
Certbot       = Tool that automatically:
                - Requests certificate from Let's Encrypt
                - Installs it in Nginx
                - Renews it every 90 days
```

```
Your Server → Certbot → Let's Encrypt → Free Certificate
                ↑
         Runs on your server
         Handles everything automatically
```

### Certificate Files in Nginx:
```
fullchain.pem  = Certificate + Public Key (safe to share)
privkey.pem    = Your Private Key (NEVER share this!)
```

---

## 11. Nginx Configuration — Manual OSI

> **Key Insight:**
> - Browsing google.com = OSI happens **automatically** (you are the passenger ✈️)
> - Nginx configuration = You **manually configure** each OSI layer (you are the pilot ✈️)

### OSI Layer Mapping:

| OSI Layer | Google.com (Automatic) | Your Nginx Config (Manual) |
|-----------|----------------------|---------------------------|
| 7. Application | Chrome handles HTTP/2 auto | `location / { }` |
| 6. Presentation | Google's cert auto installed | `ssl_certificate` path |
| 5. Session | Browser manages auto | `server_name` |
| 4. Transport | TCP auto picks port 443 | `listen 443` |
| 3. Network | Google's routers handle routing | Your server IP |
| 2. Data Link | Auto handled by hardware | Auto handled by hardware |
| 1. Physical | Google's cables | Your hosting provider's cables |

### Annotated Nginx Config:
```nginx
server {

    # ── TRANSPORT LAYER (Layer 4) ──────────────────
    listen 80;           # Port 80  = HTTP
    listen 443 ssl http2;# Port 443 = HTTPS + HTTP/2 enabled!

    # ── SESSION LAYER (Layer 5) ────────────────────
    server_name yoursite.com www.yoursite.com;
    # Identifies which website, manages sessions

    # ── PRESENTATION LAYER (Layer 6) ──────────────
    ssl_certificate     /etc/letsencrypt/live/yoursite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yoursite.com/privkey.pem;
    # Enables TLS encryption using Let's Encrypt certificate

    # ── APPLICATION LAYER (Layer 7) ────────────────
    location / {
        root /var/www/html;   # Where website files live
        index index.html;     # First file to serve
    }
    # Handles HTTP GET requests

    # ── FORCE HTTPS ────────────────────────────────
    if ($scheme = http) {
        return 301 https://$host$request_uri;
    }
    # Redirect HTTP → HTTPS (force encryption)
}
```

### Quick Reference Map:
```
What you learned     →    Nginx config line
─────────────────────────────────────────────
Port 443 (HTTPS)     →    listen 443 ssl
Port 80  (HTTP)      →    listen 80
HTTP/2               →    listen 443 ssl http2
SSL/TLS Encryption   →    ssl_certificate
Let's Encrypt cert   →    /etc/letsencrypt/...
Session management   →    server_name
HTTP GET handler     →    location / { }
Force HTTPS          →    return 301 https://
```

---

## Quick Revision Summary

```
Network Message     → Data + Headers traveling between computers
Client-Server       → Browser (asks) ↔ Server (responds)
OSI Model           → 7 layer journey map of every message
Physical Layer      → Dumb signal sender (bits only)
Data Link Layer     → Smart frame handler (MAC + error check)
Encapsulation       → Each layer wraps message going DOWN
Decapsulation       → Each layer unwraps message going UP
HTTP                → Plain text, port 80, not safe
HTTPS               → Encrypted, port 443, safe 🔒
HTTP/2              → Multiplexing, binary, server push, faster
Asymmetric Encrypt  → Public/Private key pair (slow, secure)
Symmetric Encrypt   → Session key both sides (fast)
SSL Certificate     → Proof of identity + contains public key
Let's Encrypt       → Free certificate authority
Certbot             → Auto installs + renews certificate
Nginx Config        → You manually configuring OSI layers
```

---

*Learned through practical flow: browser → DNS → TCP → OSI → HTTP/2 → SSL/TLS → Nginx*
