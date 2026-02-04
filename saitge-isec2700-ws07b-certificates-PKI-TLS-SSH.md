# Workshop 07b — **Certificates, PKI, TLS, and SSH in a GNS3 Lab**

**Theme:** *Trust, Identity, and Secure Connections (Windows + Ubuntu Clients → Ubuntu Server)*  
**Platforms:** GNS3, Windows 11 Client, Ubuntu Desktop Client, Ubuntu Server (services)  
**Instructor Role:** Provide GNS3 topology diagram + distribute hostnames/IP plan (below)  
**Student Role:** Build trust correctly (not “click through warnings”), demonstrate TLS and SSH security, and explain tradeoffs.

***

## 0) The SAITGE Story (Case Study Context)

You are a junior member of the **Strait Area Campus Information Technology Generalist (SAITGE) DevOps team**. Your lab is being expanded to support secure internal services used in networking and DevOps training. The team’s security goal is to move from “it works” to “it’s verifiably trusted.”

In this workshop, you will simulate a real SAITGE requirement:

1.  **Ubuntu Server** hosts an internal web service (Nginx) over **HTTPS (TLS)**.
2.  **Windows 11** and **Ubuntu Desktop** clients must connect without certificate warnings by correctly installing trust in each OS certificate store.
3.  Students also configure **SSH key authentication** to the Ubuntu server and relate it to public-key trust models.

You will learn how **PKI + TLS certificates** relate to **SSH keys**, and why “self-signed” is different from “CA-signed.”

***

# 1) Comprehensive Overview: Certificates, PKI, TLS, and SSH

## 1.1 What a certificate *is* (and what it is not)

A **TLS certificate** is an identity document that binds a **public key** to an identity (hostname/service/org). It is used to authenticate and encrypt traffic in TLS connections. [\[geeksforgeeks.org\]](https://www.geeksforgeeks.org/devops/how-to-install-an-ssl-certificate-on-azure/), [\[gist.github.com\]](https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc)

A certificate **is not** encryption by itself. Encryption happens during the **TLS handshake**, which uses certificates + keys to negotiate session encryption.

***

## 1.2 PKI (Public Key Infrastructure) in one picture

PKI is the *system of trust* that makes certificates meaningful.

*   **Root CA** (trusted anchor) → signs **Intermediate CA(s)** → signs **Server certificates**.
*   Clients trust the Root CA, and that trust extends down the chain. [\[geeksforgeeks.org\]](https://www.geeksforgeeks.org/devops/how-to-install-an-ssl-certificate-on-azure/), [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

### Why this matters

If your server certificate is signed by a **CA** already trusted by the client OS/browser, your HTTPS connection is trusted automatically. With **self-signed**, it’s not.

***

## 1.3 TLS (Transport Layer Security): what it guarantees

TLS provides:

*   **Confidentiality**: encrypts traffic
*   **Integrity**: detects tampering
*   **Authentication**: validates server identity via certificates [\[gist.github.com\]](https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc), [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

Microsoft specifically notes that successful TLS handshake validation is essential for clients/agents, and that clients must trust the server certificate using the OS certificate store to avoid security errors. [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

## 1.4 SSH: how it’s related (and how it differs)

SSH also uses public key cryptography, but the trust model is different:

*   **TLS/PKI trust** is usually built by **trusted CAs** and certificate chains.
*   **SSH trust** is often:
    *   **Host trust** via `known_hosts` (first-seen trust / TOFU)
    *   **User authentication** via public keys stored in `authorized_keys`

This workshop uses SSH as a comparison point: keys prove identity without being transmitted, and trust is established by installing public keys (similar principle to trusting a CA certificate). [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/), [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

## 1.5 CA-signed vs Self-signed vs “Local CA” (Use cases students must know)

### A) **Public CA-signed certificates** (e.g., DigiCert, Let’s Encrypt)

**Use when:** Public-facing services, production systems, broad users.  
**Benefit:** Users already trust the CA; no manual trust deployment required.  
**Why:** Trust is anchored in client OS/browser CA stores. [\[geeksforgeeks.org\]](https://www.geeksforgeeks.org/devops/how-to-install-an-ssl-certificate-on-azure/), [\[gist.github.com\]](https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc)

### B) **Self-signed certificates**

**Use when:** labs, isolated/internal testing, short-lived dev setups.  
**Benefit:** Fast and free.  
**Cost:** Every client must be manually configured to trust it (or warnings/errors). [\[geeksforgeeks.org\]](https://www.geeksforgeeks.org/devops/how-to-install-an-ssl-certificate-on-azure/), [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

### C) **Local CA (Internal CA) — “enterprise PKI”**

**Use when:** Internal services at scale (campus systems), where you control clients.  
**Benefit:** You install the internal CA once, then issue many server certs that all clients trust. Ubuntu explicitly describes installing a local CA cert into the trust store for enterprise environments. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

> **Critical thinking hook:** Self-signed is okay for *one* service. Internal CA becomes essential when you have *many* services.

***

## 1.6 The certificate store (why it matters)

A certificate is trusted only if the OS/application trusts the issuer.

*   On **Ubuntu**, adding a CA cert to the trust store involves placing a `.crt` in `/usr/local/share/ca-certificates/` then running `update-ca-certificates`. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)
*   On **Windows**, trust is managed in certificate stores (e.g., “Trusted Root Certification Authorities”). Microsoft emphasizes that agents and clients must trust the certificate in the OS store to avoid TLS validation errors. [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

# 2) Lab Environment — GNS3 Topology & IPv4 Plan

## 2.1 Instructor-provided topology diagram

Instructor will provide the GNS3 topology diagram showing:

*   **Windows 11 Client**
*   **Ubuntu Desktop Client**
*   **Ubuntu Server** (Nginx + SSH)
*   **Optional Router/Switch node** in GNS3 for realism

Students should label nodes and interfaces in the diagram.

***

## 2.2 Recommended addressing plan (IPv4)

Use a single lab subnet; instructor may adjust for your environment.

**Lab Subnet:** `172.16.70.0/24`  
**Gateway (if router used):** `172.16.70.1`

*   **Ubuntu Server (web + SSH):** `172.16.70.10`
    *   Hostname: `web.saitge.lab`
*   **Windows 11 client:** `172.16.70.20`
*   **Ubuntu Desktop client:** `172.16.70.30`

**DNS note:** If you do not have internal DNS, students will use `hosts` files on both clients to resolve `web.saitge.lab` to `172.16.70.10`.

***

# 3) Workshop Tasks (Integrated: PKI → TLS → Trust Stores → SSH Comparison)

## Part A — Build the secure HTTPS service on Ubuntu Server (Nginx + TLS)

### A1) Install Nginx and OpenSSL (Ubuntu Server)

On Ubuntu Server:

```bash
sudo apt update
sudo apt install -y nginx openssl ca-certificates
```

Nginx’s official guidance shows how to configure HTTPS using `ssl_certificate` and `ssl_certificate_key` and to listen on 443. [\[knowledge....gicert.com\]](https://knowledge.digicert.com/tutorials/create-csr-using-openssl-and-install-your-ssl-certificate-on-an-apache-server)

***

### A2) Generate a self-signed certificate **with SAN**

Modern clients require SAN entries; OpenSSL supports adding SAN via `-addext`. [\[jeffbrown.tech\]](https://jeffbrown.tech/azure-devops-service-connection/), [\[stackoverflow.com\]](https://stackoverflow.com/questions/77683284/how-to-clone-a-repo-with-a-azure-devops-link-and-an-ssh-key)

Create a TLS key/cert pair:

```bash
sudo mkdir -p /etc/ssl/saitge
cd /etc/ssl/saitge

sudo openssl req -nodes -x509 -sha256 -newkey rsa:4096 \
  -keyout web.saitge.lab.key \
  -out web.saitge.lab.crt \
  -days 365 \
  -subj "/CN=web.saitge.lab" \
  -addext "subjectAltName=DNS:web.saitge.lab,IP:172.16.70.10,DNS:localhost"
```

Lock down the private key:

```bash
sudo chmod 600 /etc/ssl/saitge/web.saitge.lab.key
```

Nginx documentation emphasizes the private key is sensitive and should be protected with restricted access. [\[knowledge....gicert.com\]](https://knowledge.digicert.com/tutorials/create-csr-using-openssl-and-install-your-ssl-certificate-on-an-apache-server)

***

### A3) Configure Nginx to use TLS

Create a site config:

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

This aligns directly with Nginx’s HTTPS server configuration model. [\[knowledge....gicert.com\]](https://knowledge.digicert.com/tutorials/create-csr-using-openssl-and-install-your-ssl-certificate-on-an-apache-server)

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/web.saitge.lab /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

***

### A4) Verify TLS locally on the server

```bash
curl -vk https://localhost/
```

You will likely see a trust warning because the cert is self-signed (expected at this stage). [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

***

## Part B — Make clients resolve the hostname correctly (DNS/hosts)

If no internal DNS exists, configure hosts files:

### B1) On Windows 11 client (PowerShell as admin)

Edit:
`C:\Windows\System32\drivers\etc\hosts`

Add:

    172.16.70.10  web.saitge.lab

### B2) On Ubuntu Desktop client

Edit:

```bash
sudo nano /etc/hosts
```

Add:

    172.16.70.10  web.saitge.lab

**Critical thinking:** Why is hostname matching important in TLS?  
Because certificate validation checks that the hostname you typed is present in the certificate identity (SAN). [\[jeffbrown.tech\]](https://jeffbrown.tech/azure-devops-service-connection/), [\[stackoverflow.com\]](https://stackoverflow.com/questions/77683284/how-to-clone-a-repo-with-a-azure-devops-link-and-an-ssh-key)

***

## Part C — Trust the certificate properly (Certificate stores)

### C1) Export or copy the public certificate from server to clients

Copy the public certificate file `web.saitge.lab.crt` to each client (SCP, shared folder, or GNS3 file share).

***

### C2) Install trust on Ubuntu Desktop

Ubuntu’s official guidance: copy `.crt` into `/usr/local/share/ca-certificates/` and run `update-ca-certificates`. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

On Ubuntu Desktop:

```bash
sudo cp web.saitge.lab.crt /usr/local/share/ca-certificates/web.saitge.lab.crt
sudo update-ca-certificates
```

Verify:

```bash
curl -v https://web.saitge.lab/
```

If the trust store is correct, you should not see “unknown authority” errors. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

***

### C3) Install trust on Windows 11

Microsoft’s agent guidance emphasizes the OS store must trust the server certificate to avoid TLS security errors. [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

On Windows 11:

1.  Double-click `web.saitge.lab.crt` (or `.cer` if you exported)
2.  **Install Certificate**
3.  Choose **Local Machine**
4.  Place in **Trusted Root Certification Authorities**

Verify TLS handshake:

```powershell
Invoke-WebRequest -Uri https://web.saitge.lab/ -UseBasicParsing
```

Even if an application returns an auth error, a clean handshake indicates trust is correctly installed (this is exactly how Microsoft recommends validating self-signed trust for agents). [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

## Part D — SSH keys: compare and connect the trust model (optional but recommended)

### D1) Enable SSH key login on Ubuntu Server (if not already)

On Ubuntu Server:

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

### D2) Create SSH keys on Windows 11 and push public key to Ubuntu Server

On Windows 11:

```powershell
ssh-keygen -t rsa-sha2-256 -b 4096
```

Push the public key to Ubuntu Server (PowerShell):

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub |
ssh student@172.16.70.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Fix permissions:

```powershell
ssh student@172.16.70.10 "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

**Critical thinking:**

*   In TLS, you installed trust into a *certificate store.*
*   In SSH, you installed trust by placing your public key into *authorized\_keys.*  
    Both are “trust distribution,” just implemented differently. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/), [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

# 4) Required Evidence (Students must capture outputs)

Students should record:

1.  Screenshot of `curl -vk https://web.saitge.lab/` **before** trust is installed (shows warning). [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)
2.  Screenshot of successful `curl -v https://web.saitge.lab/` **after** Ubuntu trust store installation. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)
3.  Screenshot of Windows certificate installed in **Trusted Root Certification Authorities** (or proof via MMC).
4.  Output of `Invoke-WebRequest https://web.saitge.lab/` showing no TLS validation failure. [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)
5.  SSH proof: `ssh student@172.16.70.10` shows no password prompt after key install (optional).

***

# 5) Reflection (Critical Thinking — required)

Write **350–500 words** answering the prompts below. Your answer will be graded for clarity, accuracy, and reasoning.

## Reflection prompts

1.  **Trust model comparison:**  
    Explain how trust is established in **PKI/TLS** versus **SSH**. In your own words, what served as the “trust anchor” in each case? [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/), [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

2.  **CA vs self-signed decision:**  
    Give two scenarios where **CA-signed** certificates are required and two where **self-signed** is acceptable. Justify with security and operational reasons (deployment scale, user trust, risk). [\[geeksforgeeks.org\]](https://www.geeksforgeeks.org/devops/how-to-install-an-ssl-certificate-on-azure/), [\[gist.github.com\]](https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc)

3.  **Local CA critical thinking:**  
    If SAITGE had 30 internal services (DevOps, monitoring, wiki, API gateway, dashboards), what breaks if each is self-signed? How would an internal CA reduce workload? [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)

4.  **SAN and identity:**  
    Why is SAN important for modern TLS validation? How could a mismatch create risk or failure? [\[jeffbrown.tech\]](https://jeffbrown.tech/azure-devops-service-connection/), [\[stackoverflow.com\]](https://stackoverflow.com/questions/77683284/how-to-clone-a-repo-with-a-azure-devops-link-and-an-ssh-key)

5.  **Security tradeoffs:**  
    Where do you draw the line between “lab convenience” and “professional security practice”? Provide one concrete example from this lab (e.g., trusting a root cert, protecting private keys, expiry). [\[knowledge....gicert.com\]](https://knowledge.digicert.com/tutorials/create-csr-using-openssl-and-install-your-ssl-certificate-on-an-apache-server), [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)

***

# 6) Instructor Notes (optional guidance)

*   Encourage students to observe the exact error messages before/after trust. Ubuntu trust store steps are authoritative and should be followed carefully, including `.crt` extension requirement. [\[edvaldogui...aes.com.br\]](https://edvaldoguimaraes.com.br/2024/11/25/using-ssh-with-azure-devops-a-step-by-step-guide/)
*   Reinforce that TLS success means handshake validates; application auth is separate. Microsoft explicitly uses this concept for self-hosted agents. [\[betanet.net\]](https://betanet.net/view-post/azure-devops-ssh-configuration-6497)
*   Use Nginx TLS config as the canonical example of cert/key binding. [\[knowledge....gicert.com\]](https://knowledge.digicert.com/tutorials/create-csr-using-openssl-and-install-your-ssl-certificate-on-an-apache-server)

***

## Quick clarifying questions (so I tailor the lab “just right”)

1.  Will students run the **Windows 11 client inside GNS3** (as a VM), or on their physical laptops connected to the GNS3 lab network?
2.  Do you want the Ubuntu Server to host **only Nginx**, or also a second service (e.g., Git over HTTPS, simple API endpoint) to reinforce “many services → internal CA”?

If you answer those, I can adapt the workshop to your exact topology constraints and add a “bonus internal CA mini-capstone” that naturally leads into Workshop 08.
