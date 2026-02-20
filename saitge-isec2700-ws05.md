# **Workshop 05 — Risk Mitigation, Security Policies, and Disaster Recovery Planning (Domain & File Services)**

**Course:** ISEC 2700 – Introduction to Information Security Practices
**Workshop Type:** Hands-On (Assessed Workshop)
**Estimated Time:** 1–2 hours
**Weight:** **5% (Workshop Assessment)**
**Primary Learning Outcome:** **LO2 – Assess information security risks for a small business environment**
**Supports:** LO1 (identify issues), LO3 (controls planning)
**Case Study Source:** **OSYS2020 — Windows Domain, File & Storage Services**

---

## 1. Assignment Details

| Field             | Information                                                                      |
| ----------------- | -------------------------------------------------------------------------------- |
| Workshop Title    | Workshop 05 — Risk Mitigation, Security Policies, and Disaster Recovery Planning |
| Delivery Mode     | In-class / Guided Lab (GNS3 + documentation)                                     |
| Prerequisites     | Workshop 04 risk matrix; CIA fundamentals; security domains                      |
| Tools / Resources | OSYS2020 domain diagram, file/storage scenario description, templates provided   |
| Submission        | Single PDF or DOCX (all deliverables + reflection)                               |

---

## 2. Overview, Purpose, and Objectives

### Overview

In this workshop, you will take the **security risks identified in Workshop 04** and apply them to a **Windows domain environment** that provides **file and storage services** to users.

You will decide:

* which risks must be mitigated,
* which policies enforce safe behaviour,
* and how the organization recovers when file or directory services fail.

---

### Purpose (Why this matters)

File services and Active Directory are **high-impact assets**:

* A compromised domain affects *every system*
* File server outages directly affect **availability**
* Poor access control leads to **data breaches**

This workshop mirrors how real IT and security teams plan **risk mitigation and recovery** for core infrastructure.

---

### Objectives

By the end of this workshop, you will be able to:

1. Select appropriate **risk treatments** for domain and file-service risks.
2. Design **security policies** that reduce misuse, misconfiguration, and compromise.
3. Create a **disaster recovery plan** for directory services and file storage.
4. Produce a professional **risk mitigation & DR package** aligned to organizational needs.

---

## 3. Learning Outcomes Addressed

### **LO2 — Assess information security risks**

You will demonstrate your ability to:

* act on risk assessment results,
* justify mitigation choices,
* and design recovery planning for critical services.

---

## 4. Case Study Context — OSYS2020 Domain & Storage Environment

Use the following **reference environment**:

### Core Services

* **Windows Server (Active Directory Domain Services)**
* **DNS and DHCP**
* **File Server (NTFS + SMB shares)**
* **User & Group-based access control**
* **Backup storage (on-prem or external)**

### Clients

* Windows 11 domain-joined workstations
* Staff and student user accounts

> ⚠️ You are not configuring servers today.
> You are **planning how to reduce risk and recover** when failures or incidents occur.

---

# 5. Instructional Planning Flow

## A) Introduction (Hook)

A staff member reports:

* They cannot access shared files
* Several folders appear encrypted
* A domain admin account logged in at 3 a.m.

Ask:

* What part of **CIA** is most affected?
* What service failure hurts the organization the most?
* What policies should already exist to limit damage?

---

## B) Assessment of Understanding (Quick Diagnostic)

Answer briefly:

1. Why is **Active Directory** considered a high-value target?
2. Name the **four risk treatment options**.
3. What is the difference between **backup** and **disaster recovery** for file services?

---

## C) Interaction (Hands-On Tasks)

---

## D) Re-Assessment (Peer Check)

Peer review another team’s work:

* Are mitigations realistic for a small business?
* Are policies enforceable with Windows tools?
* Could someone follow the DR steps during an outage?

Provide one written improvement suggestion.

---

## E) Closure and Transition (3-2-1)

* **3** controls you would prioritize for the domain
* **2** policies that reduce misuse or compromise
* **1** DR weakness you would address next

---

# 6. Workshop Tasks and Instructions

---

## **Task 1 — Select Top 5 Domain & File Service Risks**

From your Workshop 04 risk matrix, select **five risks** relevant to the domain environment.

Examples (students choose their own):

* Excessive domain admin privileges
* Weak password policy
* No tested file backups
* Ransomware exposure on file shares
* Single domain controller (SPOF)

For each risk, record:

* Risk ID / title
* Affected domain (IAM, data, detection, response)
* CIA impact
* Likelihood & impact rating

### Deliverable

**Top 5 Risk Register** (Template A)

---

## **Task 2 — Choose a Risk Treatment**

For each risk, select one:

* Mitigate
* Avoid
* Transfer
* Accept (with justification + monitoring)

> Accepting risk in a domain environment must be justified carefully.

### Deliverable

Updated Risk Register with **Treatment + Justification**

---

## **Task 3 — Build a Domain Risk Mitigation Plan**

For each mitigated risk, propose **2–3 controls**.

Examples (scaffold only):

### IAM

* Tiered admin accounts
* Group-based access (no direct user permissions)
* MFA for domain admins

### Data / File Services

* NTFS least privilege
* Separate admin and data shares
* Shadow copies for quick restores

### Detection

* Logon auditing
* File access auditing
* Alerting for admin logons

### Response

* Account disable procedures
* Share isolation steps
* Restore playbooks

### Deliverable

**Mitigation Plan** (Template B)

---

## **Task 4 — Security Policies for Domain & Storage**

Write **two security policies** that reduce domain/file risk.

Recommended choices:

1. **Access Control & Privileged Account Policy**
2. **Backup & Recovery Policy**
3. **Acceptable Use & File Storage Policy**
4. **Incident Response Policy (Account Compromise)**

Each policy must include:

* Purpose
* Scope
* MUST / MUST NOT statements
* Roles & responsibilities
* Enforcement & evidence
* Exceptions process

### Deliverable

Two 1-page structured policies

---

## **Task 5 — Disaster Recovery Planning (Domain & File Services)**

### Step 5.1 — Define RTO / RPO

Define RTO/RPO for:

* Active Directory
* File Server
* DNS/DHCP

Explain why AD often has the **tightest RTO**.

---

### Step 5.2 — Build a DR Checklist

Your checklist must include:

1. Trigger conditions (ransomware, DC failure, corruption)
2. Roles & contacts
3. Restore priority order
4. Backup sources (system state, file backups)
5. Recovery steps
6. Validation steps (user login, file access)
7. Post-incident actions

### Deliverable

Testable **DR Checklist** (Template C)

---

# 7. Final Deliverables

Submit **one document** containing:

1. Top 5 Risk Register
2. Domain Risk Mitigation Plan
3. Two Security Policies
4. Disaster Recovery Checklist
5. Reflection (300–400 words)

---

# 8. Reflection Questions

Answer all (300–400 words):

1. Which domain risk posed the greatest organizational impact, and why?
2. How did least privilege influence your mitigation decisions?
3. Which policy would be hardest to enforce in a real environment?
4. How did RTO/RPO affect domain and file recovery priorities?
5. What detection improvement would most reduce future risk?

---

# 9. Assessment Criteria (Skills-Based)

| Criteria             | Evidence of Mastery                         |
| -------------------- | ------------------------------------------- |
| Risk prioritization  | Risks clearly tied to domain services & CIA |
| Treatment decisions  | Appropriate and justified                   |
| Mitigation controls  | Practical Windows-based controls            |
| Policy quality       | Enforceable, auditable                      |
| DR planning          | Clear, testable, prioritized                |
| Professional quality | Clear, structured, realistic                |

---

## Why this is a strong redesign (instructor view)

* 🔗 **Cross-course integration** (ISEC ↔ OSYS)
* 🧠 Clear mental model for students (domains + files = high value)
* 🛡️ Perfect fit for LO2 (assessment, not config)
* 📁 Sets up later **file server hardening, backups, IR labs**

