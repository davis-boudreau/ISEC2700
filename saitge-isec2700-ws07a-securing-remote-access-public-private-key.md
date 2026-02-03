# **Workshop 07 — Secure DevOps Access with PKI, TLS, and SSH**

## Assignment Details

| Item               | Details                                                        |
| ------------------ | -------------------------------------------------------------- |
| **Course**         | ISEC 2700 – Information Systems Security                       |
| **Workshop Title** | Secure DevOps Access with PKI, TLS, and SSH                    |
| **Type**           | Hands-On Workshop                                              |
| **Delivery Mode**  | In-Class / Guided Lab                                          |
| **Estimated Time** | 1 hour                                                         |
| **Weighting**      | **Workshop Assessment (Applied Learning – Skills-Based)**      |
| **Prerequisites**  | Basic networking concepts, TCP/IP, authentication fundamentals |
| **Tools Required** | FortiClient VPN (if remote), VS Code, PuTTY, PuTTYgen          |
| **Submission**     | Screenshots + reflection                                       |

---

## **Workshop Purpose (Why This Exists)**

Modern DevOps environments **do not rely on passwords** for system-to-system or developer access.

Instead, organizations use:

* **PKI + TLS** → to trust *systems*
* **SSH keys** → to trust *people*

In this workshop, you will **experience both models** and understand:

* how trust is established,
* where cryptography is used,
* and why these approaches scale securely in real organizations.

---

## **Learning Outcomes Addressed**

* **LO3:** Identify and implement security controls to maintain confidentiality, integrity, and availability
* **LO1 (supporting):** Identify security issues related to authentication and trust
* **LO2 (supporting):** Assess the risk of weak authentication mechanisms

---

## **Workshop Storyline: The SAITGE DevOps Project**

You are a member of the **SAITGE DevOps team**, supporting a campus-wide modernization initiative.

Your team is responsible for:

* Managing source code in **Azure DevOps Server 2022**
* Enforcing **secure authentication and access control**
* Supporting **Linux infrastructure** for GNS3 labs and automation

As part of onboarding, **every team member must prove**:

1. Secure, passwordless Git access using **SSH keys**
2. Secure, passwordless Linux access using **key-based SSH**

This workshop simulates that onboarding.

---

# **Learning Journey Overview**

| Phase   | Focus                 | Security Architecture Mapping  |
| ------- | --------------------- | ------------------------------ |
| Phase 1 | Trust & cryptography  | PKI, TLS, CIA                  |
| Phase 2 | Developer identity    | IAM, least privilege           |
| Phase 3 | DevOps access         | Application & network security |
| Phase 4 | Infrastructure access | Endpoint & network security    |
| Phase 5 | Reflection            | Risk, policy, response         |

---

# **FOUNDATIONS: Security Concepts You Will Use**

## **1. Public Key Infrastructure (PKI)**

PKI is the **global trust system** that allows computers to verify identities **without pre-sharing secrets**.

PKI is built on:

* **Asymmetric cryptography**
* **Certificate Authorities (CAs)**
* **Chains of trust**

### Mental model

> PKI answers: *“How do I know this server is who it claims to be?”*

In SAITGE:

* PKI protects **HTTPS**
* PKI protects **APIs**
* PKI protects **Azure DevOps Server**

---

## **2. Digital Certificates**

A digital certificate:

* Binds an **identity** (DNS name, organization) to a **public key**
* Is **signed** by a trusted CA
* Is validated automatically by clients

When you visit Azure DevOps Server:

* Your browser checks the certificate
* The certificate enables **encrypted TLS communication**
* You never manually exchange keys

This supports:

* **Confidentiality** (encryption)
* **Integrity** (tamper detection)
* **Authentication** (server identity)

---

## **3. TLS (Transport Layer Security)**

TLS is **not just encryption** — it is a **security protocol** that enforces:

* 🔒 Confidentiality
* ✍️ Integrity
* 🧾 Authentication

TLS is why:

* Web traffic is HTTPS
* Git over HTTPS is secure
* APIs can be trusted at scale

TLS protects **data in transit**, not access rights.

---

## **4. SSH Keys: Developer Identity**

SSH keys solve a *different* problem than TLS.

SSH answers:

> *“How does this server know who I am?”*

SSH keys:

* Identify **people**, not servers
* Do **not use a CA**
* Use **explicit trust** (public keys installed manually)

This maps directly to:

* **Identity & Access Management**
* **Least Privilege**
* **Separation of Duties**

---

# **COMMON SETUP — Developer Identity**

## **Task 0 — Generate Your SSH Key Pair (Windows 11)**

Your SSH key pair is your **developer identity**.

Open **PowerShell**:

```powershell
ssh-keygen -t rsa-sha2-256 -b 4096
```

When prompted:

* Accept default file location
* **Enter a passphrase** (required by policy)

This creates:

```
id_rsa       → private key (never shared)
id_rsa.pub   → public key (safe to distribute)
```

### Why this matters

* Strong asymmetric crypto
* Keys scale better than passwords
* Passphrases protect keys at rest

---

## **Task 0.1 — Verify Key Creation**

```powershell
dir $env:USERPROFILE\.ssh
```

---

# ✅ **PART A — Case Study A**

## **Secure DevOps Access (Azure DevOps + SSH)**

### Security Policy Context

* ❌ No passwords
* ❌ No PATs
* ✅ SSH key authentication only

This enforces:

* Strong identity
* Non-repudiation
* Easy off-boarding

---

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

## Part A.2 — Test SSH Authentication to DevOps

```powershell
ssh -T wXXXXXXX@172.16.184.104
```
where wXXXXXXX is your student ID.

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

---

## Part A — Reflection Checkpoint

Answer briefly:

* Why is SSH preferred over passwords or PATs for daily Git work?
* How does your SSH key function like a digital identity?

---

# ✅ **PART B — Case Study B**

## **SAITGE Linux Infrastructure Access with PuTTY (SSH Key-Based Authentication)**

---

## **Why This Case Study Exists**

In a secure enterprise environment, **Linux servers should never rely on passwords** for interactive access.

Passwords are:

* Guessable
* Reusable
* Phishable
* Difficult to revoke cleanly at scale

Instead, SAITGE uses **SSH key-based authentication** to ensure that:

* Each administrator has a **unique cryptographic identity**
* Access can be **granted and revoked explicitly**
* Compromised passwords do not lead to lateral movement

This case study focuses on **infrastructure access**, not applications.

---

## **Security Architecture Mapping**

This activity maps directly to the **ISEC 2700 Security Architecture Model**:

| Architecture Element                   | How It Appears Here                         |
| -------------------------------------- | ------------------------------------------- |
| **Identity & Access Management (IAM)** | SSH keys identify the user                  |
| **Endpoints**                          | Windows workstation + Linux server          |
| **Network Security**                   | SSH over TCP 22                             |
| **CIA**                                | Confidentiality & Integrity of admin access |
| **Least Privilege**                    | Per-user access via authorized keys         |
| **Defense in Depth**                   | Passwords removed from attack surface       |

---

## **Scenario**

The SAITGE DevOps team manages Linux servers used for:

* GNS3 labs
* Automation runners
* Monitoring and telemetry tools

You must securely access a Linux host (**GNS3 Server**) using **key-based SSH authentication**.

Replace `.XXX` with your assigned desk number.
Example: Desk-121 → `172.16.184.121`

```
gns3@172.16.184.XXX
```

---

## **Access Requirements (Policy Driven)**

✔ Passwordless Linux login
✔ Windows-native tooling (PuTTY)
✔ Passphrase-protected private key
✔ Pageant for usability (with awareness of tradeoffs)

---

## **Part B.1 — Push Your Public Key to the Linux Server**

At this stage, you are performing **explicit trust establishment**.

Unlike PKI:

* There is **no Certificate Authority**
* Trust is created by **manually installing a public key**

### Command (PowerShell)

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub |
ssh gns3@172.16.184.XXX "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

> Replace `XXX` with your desk number.

### Fix Required Permissions

```powershell
ssh gns3@172.16.184.XXX "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Why permissions matter

SSH will **refuse to trust keys** if:

* `.ssh` is writable by others
* `authorized_keys` is not protected

This enforces **integrity** of authentication data.

### Concept Tie-In

This step:

* Links **your identity (key)** to **a specific account**
* Implements **least privilege**
* Eliminates password-based attack vectors

---

## **Part B.2 — Verify Access Using OpenSSH**

Before switching tools, verify that authentication works.

```powershell
ssh gns3@172.16.184.XXX
```

### Expected Result

✔ No Linux password prompt
✔ Passphrase prompt only
✔ Successful shell access

Exit the session once verified.

### Why this matters

This confirms:

* Key installation is correct
* Password authentication is not required
* The server trusts **only your key**

---

## **Part B.3 — Convert the Private Key for PuTTY**

Windows environments often standardize on **PuTTY**, which uses the `.ppk` key format.

This is a **format conversion only** — the cryptography does not change.

### PuTTYgen Steps

1. Open **PuTTYgen**
2. Click **Load**
3. Change file filter to **All Files**
4. Load:

   ```
   C:\Users\<you>\.ssh\id_rsa
   ```
5. Enter your passphrase
6. Click **Save private key**
7. Save as:

   ```
   id_rsa.ppk
   ```

### Security Note

* Your **private key never leaves your machine**
* Only the **format** changes
* The passphrase remains enforced

---

## **Part B.4 — Configure PuTTY Session**

### Authentication Setup

1. Open **PuTTY**
2. Navigate to:

   ```
   Connection → SSH → Auth → Credentials
   ```
3. Select:

   ```
   id_rsa.ppk
   ```

### Session Details

| Field           | Value                 |
| --------------- | --------------------- |
| Host            | `gns3@172.16.184.XXX` |
| Port            | `22`                  |
| Connection Type | `SSH`                 |

Save the session as:

```
Workshop07-SAITGE-Linux
```

### Why this matters

Saving sessions:

* Reduces configuration errors
* Encourages consistent secure access
* Supports operational discipline

---

## **Part B.5 — Login with PuTTY**

1. Open the saved session
2. Enter your **passphrase** (once)
3. Linux shell opens
4. No password is requested ✅

### Security Interpretation

* Authentication succeeded using **asymmetric cryptography**
* Password attacks are fully removed
* Identity is cryptographically enforced

---

## **Part B.6 — Automate Access with Pageant**

Pageant is an **SSH authentication agent**.

It temporarily holds decrypted keys **in memory**.

### Steps

1. Start **Pageant**
2. Add:

   ```
   id_rsa.ppk
   ```
3. Enter passphrase once

Now:

* Open PuTTY
* Login occurs with **zero prompts**

---

### Security Discussion (Critical Thinking)

Pageant introduces a **security vs usability tradeoff**:

| Benefit         | Risk                            |
| --------------- | ------------------------------- |
| Faster access   | Keys live in memory             |
| Fewer prompts   | Requires workstation security   |
| Better workflow | Screen locking becomes critical |

**Operational expectation:**

> If Pageant is running, **lock your screen when unattended**.

---

## **What You Have Proven in Part B**

By completing this case study, you have demonstrated that you can:

* Establish **explicit trust** without a CA
* Implement **passwordless infrastructure access**
* Enforce **least privilege**
* Use **Windows-standard tooling securely**
* Reason about **operational security tradeoffs**

This directly supports **LO3** and prepares you for **incident response and access revocation scenarios** later in the course.

---

# **Final Reflection — Professional Perspective**

This reflection **cements understanding** and ties the workshop back to:

* Architecture
* Policy
* Risk
* Response

---

## **End State (What You Now Know)**

By the end of this workshop, you can:

* Explain PKI vs SSH trust
* Describe TLS’s role in CIA
* Use SSH securely in DevOps
* Reason about authentication risk
* Apply security architecture thinking

---
