# **Workshop 07 — Secure DevOps Access with PKI, TLS, and SSH**

**Course Context:** SAITGE (Strait Area Campus Information Technology Generalist)  
**Platform:** Windows 11, VS Code, Azure DevOps Server 2022  
**Theme:** *Identity, Trust, and Secure Access in a DevOps Team*

***

## **Workshop Storyline: The SAITGE DevOps Project**

You are a **member of the SAITGE DevOps team**, supporting a campus‑wide modernization project.

The team is responsible for:

*   Managing source code in **Azure DevOps Server 2022**
*   Enforcing **secure authentication practices**
*   Supporting **Linux infrastructure** used by networking, GNS3 labs, and automation pipelines

As part of onboarding, **every team member must demonstrate**:

1.  Secure access to the DevOps Git repositories using **SSH keys** (no passwords, no PATs)
2.  Secure, passwordless access to Linux servers using **PuTTY and key‑based authentication**

This workshop simulates that onboarding process.

***

# **Learning Journey Overview**

### Phase 1 — Foundations (Why security works)

You learn **PKI, certificates, TLS**, and how trust is established.

### Phase 2 — Developer Identity (Who you are)

You generate and protect **SSH keys** on Windows 11.

### Phase 3 — Case Study A (DevOps access)

You join the **SAITGE DevOps repo** using VS Code and SSH.

### Phase 4 — Case Study B (Infrastructure access)

You securely access a Linux server using **PuTTY + Pageant**, converting your key to `.ppk`.

### Phase 5 — Reflection (Professional thinking)

You explain *why* these choices matter in a real organization.

***

# **FOUNDATIONS: Security Concepts You Will Use**

## 1. Public Key Infrastructure (PKI)

PKI is the system that allows identities to be trusted using cryptography.

In PKI:

*   A **public key** identifies an entity
*   A **private key** proves ownership
*   A **Certificate Authority (CA)** vouches for identity

In SAITGE:

*   PKI secures HTTPS, APIs, and internal services
*   Azure DevOps Server uses PKI‑secured TLS for its web and Git endpoints

***

## 2. Digital Certificates

A digital certificate:

*   Binds an identity to a public key
*   Is signed by a trusted CA
*   Enables trust at scale

When you browse to Azure DevOps Server 2022:

*   TLS certificates prove the server is legitimate
*   Encryption protects credentials and data in transit

***

## 3. TLS Encryption

TLS provides:

*   **Confidentiality** (encryption)
*   **Integrity** (tamper detection)
*   **Authentication** (certificate verification)

This is why:

*   Azure DevOps web UI is HTTPS
*   Git over HTTPS is secure
*   REST APIs are protected

***

## 4. SSH Keys as “Developer Certificates”

SSH keys are similar to certificates but simpler:

*   No CA required
*   Trust is established by **explicitly installing public keys**

In SAITGE:

*   SSH keys identify *developers*
*   SSH keys replace passwords for Git and Linux access
*   Private keys never leave the user’s machine

***

# **COMMON SETUP (Both Case Studies)**

## Task 0 — Generate an SSH Key Pair on Windows 11

You are issued a workstation and must create your **developer identity**.

Open **PowerShell**:

```powershell
ssh-keygen -t rsa-sha2-256 -b 4096
```

When prompted:

*   File location → press **Enter**
*   Enter a **passphrase** (required by SAITGE policy)

This creates:

    C:\Users\<you>\.ssh\id_rsa       (private key)
    C:\Users\<you>\.ssh\id_rsa.pub   (public key)

### Why this matters

*   RSA SHA‑2 is supported by Azure DevOps Server 2022
*   Passphrases protect keys at rest
*   Keys scale better than passwords

***

## Task 0.1 — Verify Key Creation

```powershell
dir $env:USERPROFILE\.ssh
```

You should see:

*   `id_rsa`
*   `id_rsa.pub`

***

# ✅ **PART A — Case Study A**

## *SAITGE DevOps Repository Access with VS Code*

### Scenario

You are assigned to the **SAITGE‑CampusAutomation** project hosted in **Azure DevOps Server 2022**.

Security policy:

*   ❌ No passwords
*   ❌ No Personal Access Tokens
*   ✅ SSH key authentication only

***

## Part A.1 — Upload Your Public Key to Azure DevOps

1.  Sign in to **Azure DevOps Server**
2.  Click your profile (top‑right)
3.  Go to:
    **User Settings → SSH Public Keys**
4.  Click **Add**
5.  Paste your public key:

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub
```

Save the key.

### Security context

You are registering your **developer identity**, not a secret.

***

## Part A.2 — Test SSH Authentication to DevOps Server (172.16.184.104)

```powershell
ssh -T wXXXXXXX@172.16.184.104
```

Expected:

*   First‑time host trust prompt → `yes`
*   Message indicating shell access is not supported

✅ Authentication succeeded

***

## Part A.3 — Clone the SAITGE Repo Using VS Code

1.  Open **VS Code**
2.  `Ctrl + Shift + P`
3.  Select **Git: Clone**
4.  Paste the SSH clone URL from Azure DevOps:

```text
IPv4:
ssh://172.16.184.104:22/DefaultCollection/Capstone2026/_git/Capstone2026
```

5.  Choose a workspace folder

Verify:

```powershell
git remote -v
```

SSH should be shown.

***

## Part A.4 — Commit and Push a Test Change

Create:

    workshop07_partA.txt

Add content:

    SAITGE DevOps onboarding – SSH authentication verified.

Commit and push:

```powershell
git add .
git commit -m "Workshop 07 Part A: SAITGE SSH access verified"
git push
```

***

## Part A — Reflection Checkpoint

Answer briefly:

*   Why is SSH preferred over passwords or PATs for daily Git work?
*   How does your SSH key function like a digital identity?

***

# ✅ **PART B — Case Study B**

## *SAITGE Linux Infrastructure Access with PuTTY*

### Scenario

The SAITGE DevOps team manages Linux servers for:

*   GNS3 labs
*   Automation runners
*   Monitoring tools

You must access a Linux host:

    gns3@172.16.184.154

Requirements:

*   Passwordless Linux login
*   PuTTY (Windows standard)
*   Passphrase protected key
*   Pageant for usability

***

## Part B.1 — Push Your Public Key to Linux (PowerShell)

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub |
ssh gns3@172.16.184.154 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Fix permissions:

```powershell
ssh gns3@172.16.184.154 "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Concept tie‑in

This is **explicit trust establishment**—no CA involved.

***

## Part B.2 — Verify with OpenSSH

```powershell
ssh gns3@172.16.184.154
```

Expected:

*   No Linux password prompt
*   Passphrase prompt only

Exit the session.

***

## Part B.3 — Convert the Private Key for PuTTY

PuTTY uses `.ppk` format.

### PuTTYgen Steps

1.  Open **PuTTYgen**
2.  Click **Load**
3.  Select **All Files**
4.  Load:
        C:\Users\<you>\.ssh\id_rsa
5.  Enter passphrase
6.  Click **Save private key**
7.  Save as:
        id_rsa.ppk

***

## Part B.4 — Configure PuTTY

1.  Open **PuTTY**
2.  Navigate to:
    **Connection → SSH → Auth → Credentials**
3.  Select:
        id_rsa.ppk

Session setup:

*   Host: `gns3@172.16.184.154`
*   Port: `22`
*   Type: `SSH`

Save session:

    Workshop07-SAITGE-Linux

***

## Part B.5 — Login with PuTTY

*   Open the saved session
*   Enter passphrase (once)
*   Linux shell opens
*   No password requested ✅

***

## Part B.6 — Automate Passphrase with Pageant

1.  Start **Pageant**
2.  Add:
        id_rsa.ppk
3.  Enter passphrase once

Now:

*   Open PuTTY session
*   Login occurs with **zero prompts**

### Security discussion

*   Pageant trades convenience for memory‑resident key material
*   Lock your screen when Pageant is running

***

# **Final Reflection — Professional Perspective**

Write a **short narrative (300–400 words)** answering:

1.  How PKI and TLS secure Azure DevOps Server differently than SSH secures Linux access.
2.  Why SSH keys are more scalable and secure than passwords in a DevOps team.
3.  The security tradeoffs of:
    *   No passphrase
    *   Passphrase only
    *   Passphrase + Pageant
4.  What policies SAITGE should enforce when:
    *   A student graduates
    *   A laptop is lost
    *   A server is decommissioned
5.  One concept that “clicked” for you during this workshop.

***

## **End State: What You’ve Achieved**

✅ You understand **why** trust works  
✅ You created a secure **developer identity**  
✅ You accessed **Azure DevOps Server 2022** securely with VS Code  
✅ You accessed **Linux infrastructure** securely with PuTTY  
✅ You practiced **real SAITGE DevOps workflows**

***