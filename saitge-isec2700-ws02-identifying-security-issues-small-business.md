## 🎯 Learning Outcome

**Identify security issues for a small business environment**

This tutorial is designed so you can **teach**, **learn**, and **assess** this outcome in a structured, practical way. It follows a professional security methodology aligned with industry best practice and scalable to your OSYS2020 / ISEC2700 teaching context.

---

# 1️⃣ Why This Outcome Matters

Small businesses are disproportionately targeted because:

* They have **limited security budgets**
* They often lack dedicated IT security staff
* They rely heavily on cloud + SaaS tools
* They underestimate risk exposure

According to the Canadian Centre for Cyber Security, small organizations face increasing ransomware, phishing, and supply-chain threats.

The goal is not to turn students into auditors —
It is to train them to **recognize risk patterns systematically**.

---

# 2️⃣ Step 1 – Define the Small Business Context

Before identifying issues, define:

### 📌 Business Profile Example

* 25 employees
* 1 Windows Server
* 20 Windows 11 desktops
* 5 laptops
* Microsoft 365
* Basic ISP firewall/router
* Wi-Fi network (shared password)
* Online accounting software
* Website hosted externally

Students must understand:

> You cannot identify security issues without understanding assets, users, and data.

---

# 3️⃣ Step 2 – Identify What Needs Protection (Assets)

Security issues only exist relative to **assets**.

### 🔐 Asset Categories

| Asset Type | Examples                           |
| ---------- | ---------------------------------- |
| Physical   | Servers, laptops, networking gear  |
| Digital    | Customer database, payroll records |
| Network    | Firewall, switches, Wi-Fi          |
| Cloud      | Microsoft 365, SaaS tools          |
| Human      | Employees, administrators          |

Introduce CIA Triad:

* Confidentiality
* Integrity
* Availability

This concept originates from foundational security frameworks such as those used by National Institute of Standards and Technology (NIST).

---

# 4️⃣ Step 3 – Common Security Issues in Small Businesses

Below are structured categories students should analyze.

---

## A. 🔓 Authentication & Access Control Issues

**Common Problems:**

* Shared accounts
* Weak passwords
* No MFA
* Employees with admin rights
* No account disable process when staff leave

**Risk:** Unauthorized access, insider threats.

**Best Practice Forward:**

* Enforce MFA
* Apply least privilege principle
* Implement role-based access control
* Conduct quarterly access reviews

---

## B. 🖥️ Endpoint Security Issues

**Common Problems:**

* No centralized antivirus
* No patch management
* Outdated Windows systems
* No device encryption
* No device inventory

**Risk:** Malware, ransomware, data loss.

**Best Practice Forward:**

* Automated patch management
* Centralized endpoint protection
* Full disk encryption (BitLocker)
* Maintain asset inventory

---

## C. 🌐 Network Security Issues

**Common Problems:**

* Flat network (no VLANs)
* Default router credentials
* No firewall rules review
* Guest Wi-Fi not separated
* No logging enabled

**Risk:** Lateral movement, data exfiltration.

**Best Practice Forward:**

* VLAN segmentation
* Business-grade firewall
* Change default credentials
* Enable logging & monitoring
* Separate guest network

---

## D. ☁️ Cloud & SaaS Security Issues

**Common Problems:**

* No MFA on Microsoft 365
* No conditional access policies
* Users share cloud files publicly
* No backup of SaaS data

**Risk:** Account takeover, data leakage.

**Best Practice Forward:**

* Enforce MFA
* Disable legacy authentication
* Restrict external sharing
* Implement cloud backup solution

---

## E. 📁 Backup & Disaster Recovery Issues

**Common Problems:**

* Backup stored on same network
* No offsite copy
* No backup testing
* No documented recovery plan

**Risk:** Permanent data loss after ransomware.

**Best Practice Forward:**

* 3-2-1 backup strategy
* Offline or immutable backups
* Quarterly restore testing
* Written recovery procedures

---

## F. 👥 Human & Policy Issues

**Common Problems:**

* No security awareness training
* No acceptable use policy
* No incident response plan
* No phishing simulations

**Risk:** Social engineering attacks.

**Best Practice Forward:**

* Annual awareness training
* Written policies
* Simple incident response playbook
* Phishing training campaigns

---

# 5️⃣ Structured Method to Meet the Learning Outcome

To ensure students truly *identify issues*, use this 5-step framework:

---

## 🔍 Step 1 – Map Assets

Create:

* Asset inventory table
* Network diagram
* Data flow diagram

---

## 🔎 Step 2 – Identify Threats

Examples:

* Ransomware
* Phishing
* Insider misuse
* Hardware theft
* Data breach

---

## ⚠️ Step 3 – Identify Vulnerabilities

Ask:

* What is misconfigured?
* What is outdated?
* What is missing?
* What is unmanaged?

---

## 📊 Step 4 – Assess Risk

Risk = Threat × Vulnerability × Impact

Use a simple risk matrix:

| Likelihood | Impact | Risk Level |
| ---------- | ------ | ---------- |
| High       | High   | Critical   |
| High       | Medium | High       |
| Medium     | Medium | Moderate   |

---

## 🛠 Step 5 – Recommend Controls

Controls fall into:

* Administrative (policies)
* Technical (firewall, MFA)
* Physical (locked server room)

This aligns with risk management approaches promoted by frameworks like those referenced by International Organization for Standardization (ISO 27001).

# 9️⃣ Final Summary Framework

To identify security issues in a small business:

1. Understand the business context
2. Inventory assets
3. Identify threats
4. Identify vulnerabilities
5. Evaluate risk
6. Recommend controls
7. Prioritize improvements
