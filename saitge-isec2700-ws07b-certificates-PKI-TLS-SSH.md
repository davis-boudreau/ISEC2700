# Workshop 07b — **Certificates, PKI, TLS, and SSH in a GNS3 Lab**

**Theme:** *Trust, Identity, and Secure Connections (Windows + Ubuntu Clients → Ubuntu Server)*
**Platforms:** GNS3, Windows 11 Client, Ubuntu Desktop Client, Ubuntu Server (services)
**Student Goal:** Build trust correctly (not “click through warnings”), demonstrate TLS and SSH security, and explain tradeoffs.

---

## 1) Where this fits in the Security Architecture Model

This workshop reinforces these architecture areas:

* **Identity & Access Management (IAM):** who is trusted and why
* **Network Security:** secure transport (TLS/SSH)
* **Application Security:** service configuration (Nginx TLS)
* **Data Security:** protection of data in transit
* **Detection/Response (preview):** what evidence proves trust (handshake success, warnings removed)

**Primary Learning Outcome:** **LO3** (implement controls)
**Supports:** LO1 (identify issues like “untrusted cert”), LO2 (assess risk of weak trust)

---

## 2) The SAITGE Story (Case Study Context)

You are a junior member of the **SAITGE DevOps team**. Your lab is being expanded to support internal services used in networking and DevOps training. The goal is to move from:

> “It works” → “It’s verifiably trusted.”

In this workshop you simulate a real SAITGE requirement:

1. Ubuntu Server hosts an internal web service (Nginx) over **HTTPS (TLS)**.
2. Windows 11 and Ubuntu Desktop clients must connect **without certificate warnings** by correctly installing trust into each OS trust store.
3. Students configure **SSH key authentication** and compare SSH trust to PKI trust.

---

# 3) Background Knowledge (Short, High-Value Concepts)

## 3.1 What a certificate *is* (and what it is not)

A **TLS certificate** binds a **public key** to an identity (hostname/service). It helps a client verify it connected to the right server, then TLS encrypts the traffic. A certificate itself is not “the encryption” — encryption occurs after negotiation in the TLS handshake.

### Key point students must remember

✅ **TLS trust is about “server identity validation.”**
✅ **Encryption is the result of a successful handshake.**

---

## 3.2 PKI in one picture (the trust chain)

PKI is the trust system behind certificates:

* **Root CA** (trusted anchor) → signs **Intermediate CA(s)** → sign **Server certificates**
* Clients trust the **root**, and trust extends down the chain.

In this lab, you are doing a simplified version:

* You create a certificate on the server
* You manually install trust on clients (like a small internal enterprise)

---

## 3.3 Why hostname identity (SAN) matters

Modern clients validate certificate identity using **Subject Alternative Name (SAN)**. If the hostname you type isn’t in the certificate SAN list, clients treat it as “wrong server,” even if encryption is possible.

**Rule:**

> The URL hostname must match the certificate SAN.

This is the most common reason students see warnings even after “installing the cert.”

---

## 3.4 Certificate stores (what “trust” really means)

A certificate is trusted only if the **issuer** is trusted by the OS/application.

* **Ubuntu**: add CA certs under `/usr/local/share/ca-certificates/` then run `update-ca-certificates` ([documentation.ubuntu.com][1])
* **Windows**: install into **Trusted Root Certification Authorities** (Local Machine) using MMC/Certificate Import Wizard ([Microsoft Learn][2])

---

## 3.5 TLS vs SSH (related math, different trust)

Both use public key cryptography, but trust differs:

* **TLS/PKI**: clients trust a CA chain (or you install a trusted root manually)
* **SSH**:

  * Host trust via `known_hosts` (first-seen trust / TOFU)
  * User authentication via public keys in `authorized_keys`

---

# 4) Lab Environment — Topology & IP Plan

## 4.1 Instructor-provided topology diagram

Instructor provides the GNS3 topology with:

* Windows 11 Client
* Ubuntu Desktop Client
* Ubuntu Server (Nginx + SSH)
* Optional router/switch node

## 4.2 Addressing Plan

* Subnet: `192.168.100.0/24`
* Gateway: `192.168.100.1`

| Node                      | IP               | Hostname         |
| ------------------------- | ---------------- | ---------------- |
| Ubuntu Server (web + SSH) | `192.168.100.10` | `web.saitge.lab` |
| Windows 11 Client         | `192.168.100.20` | —                |
| Ubuntu Desktop Client     | `192.168.100.30` | —                |

If no DNS exists, you will use `hosts` files.

---

# 5) Workshop Tasks (PKI → TLS → Trust Stores → SSH Comparison)

## Part A — Build HTTPS service on Ubuntu Server (Nginx + TLS)

### A1) Install Nginx + OpenSSL

On Ubuntu Server:

```bash
sudo apt update
sudo apt install -y nginx openssl ca-certificates
```

### A2) Generate a self-signed certificate **with SAN**

Create key/cert:

```bash
sudo mkdir -p /etc/ssl/saitge
cd /etc/ssl/saitge

sudo openssl req -nodes -x509 -sha256 -newkey rsa:4096 \
  -keyout web.saitge.lab.key \
  -out web.saitge.lab.crt \
  -days 365 \
  -subj "/CN=web.saitge.lab" \
  -addext "subjectAltName=DNS:web.saitge.lab,IP:192.168.100.10,DNS:localhost"
```

Lock down the private key:

```bash
sudo chmod 600 /etc/ssl/saitge/web.saitge.lab.key
```

**Why this matters**

* The private key is the sensitive secret. If exposed, attackers can impersonate the service.
* SAN ensures modern clients accept the identity claim.

### A3) Configure Nginx to use TLS

Create config:

```bash
sudo nano /etc/nginx/sites-available/web.saitge.lab
```

Paste:

```nginx
server {
    listen 443 ssl;
    server_name web.saitge.lab;

    ssl_certificate     /etc/ssl/saitge/web.saitge.lab.crt;
    ssl_certificate_key /etc/ssl/saitge/web.saitge.lab.key;

    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        return 200 "SAITGE Lab HTTPS is working (TLS + cert)\n";
        add_header Content-Type text/plain;
    }
}

server {
    listen 80;
    server_name web.saitge.lab;
    return 301 https://$host$request_uri;
}
```

This matches NGINX’s canonical HTTPS configuration pattern (`listen 443 ssl`, `ssl_certificate`, `ssl_certificate_key`). ([nginx.org][3])

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/web.saitge.lab /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### A4) Verify TLS locally on the server (expect warnings)

```bash
curl -vk https://localhost/
```

✅ You should see **TLS negotiation** occur.
⚠️ You will see trust warnings (self-signed) — expected at this stage.

---

## Part B — Hostname resolution (DNS/hosts)

### B1) Windows 11 client (PowerShell as Admin)

Edit:
`C:\Windows\System32\drivers\etc\hosts`

Add:

```
192.168.100.10  web.saitge.lab
```

### B2) Ubuntu Desktop client

```bash
sudo nano /etc/hosts
```

Add:

```
192.168.100.10  web.saitge.lab
```

**Checkpoint (must pass before Part C):**

* From both clients, `ping web.saitge.lab` resolves to `192.168.100.10`

---

## Part C — Trust the certificate properly (trust stores)

### C1) Copy the public certificate to clients

Copy `web.saitge.lab.crt` from server to each client (SCP, shared folder, etc.).
This is safe: it’s a **public certificate**, not the private key.

> **Instructor best practice:** ensure students *never* copy the `.key`.

---

### C2) Install trust on Ubuntu Desktop (trust store)

On Ubuntu Desktop:

```bash
sudo cp web.saitge.lab.crt /usr/local/share/ca-certificates/web.saitge.lab.crt
sudo update-ca-certificates
```

Ubuntu’s documented process is to place the cert in the trust location and run `update-ca-certificates`. ([documentation.ubuntu.com][1])

Verify:

```bash
curl -v https://web.saitge.lab/
```

✅ “unknown authority” errors should be gone.

---

### C3) Install trust on Windows 11 (Trusted Root store)

Import the certificate into the **Local Machine → Trusted Root Certification Authorities** store using MMC.

Microsoft’s documentation describes using MMC and the Certificates snap-in to manage the Trusted Root store. ([Microsoft Learn][2])

Verify handshake:

```powershell
Invoke-WebRequest -Uri https://web.saitge.lab/ -UseBasicParsing
```

✅ Success indicates TLS trust and encryption are working.

---

## Part D — SSH keys: compare trust models (optional but recommended)

### D1) Enable SSH server on Ubuntu Server

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

### D2) Create SSH keys on Windows and push trust to server

```powershell
ssh-keygen -t rsa-sha2-256 -b 4096
```

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub |
ssh student@192.168.100.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

```powershell
ssh student@192.168.100.10 "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

**Critical comparison**

* TLS: you installed trust into a **certificate store**
* SSH: you installed trust into **authorized_keys**
  Both are “trust distribution,” but implemented differently.

Verify:

```powershell
ssh student@192.168.100.10
```

✅ No password prompt after key install.

---

# 6) Required Evidence (What students must capture)

1. `curl -vk https://web.saitge.lab/` **before** trust (shows warning)
2. `curl -v https://web.saitge.lab/` **after** Ubuntu trust install (no unknown authority)
3. Proof of Windows cert trust store install (MMC screenshot of the Trusted Root store)
4. Output of `Invoke-WebRequest https://web.saitge.lab/` (no TLS validation failure)
5. (Optional) SSH proof: `ssh student@192.168.100.10` showing no password prompt

---

# 7) Troubleshooting (Students should use this before asking for help)

### Problem: Still seeing certificate warning after “installing”

* Did you browse to **[https://web.saitge.lab](https://web.saitge.lab)** (hostname), not the IP?
* Does the certificate SAN include the exact hostname?
* Did you install into the **Trusted Root** store (Windows) and run `update-ca-certificates` (Ubuntu)?

### Problem: Nginx won’t reload

* Run `sudo nginx -t` and fix the line it reports
* Check file paths for `.crt` and `.key`

### Problem: Ubuntu trust didn’t update

* Confirm the file is `.crt` and in `/usr/local/share/ca-certificates/`
* Re-run `sudo update-ca-certificates` ([manpages.ubuntu.com][4])

---

# 8) Reflection (350–500 words)

1. Compare trust anchors: PKI/TLS vs SSH (what is trusted in each?)
2. CA-signed vs self-signed: two scenarios for each, justify operationally
3. Local CA at scale: what breaks with 30 self-signed services?
4. SAN importance: what fails and what risk does mismatch introduce?
5. Lab convenience vs professional practice: one concrete example (keys, expiry, trust stores)

---

## Instructor Notes (optional)

* Emphasize: TLS handshake success ≠ application authorization (separate layers)
* Reinforce: NGINX HTTPS directives and port 443 usage are canonical ([nginx.org][3])
* Reinforce: Ubuntu trust store procedure is authoritative ([documentation.ubuntu.com][1])
* Reinforce: Windows Trusted Root store procedures are authoritative ([Microsoft Learn][2])

---