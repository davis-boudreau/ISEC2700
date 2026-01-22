# 🔐 Workshop — Secure Remote Access with VPNs (SSL-VPN & IPsec)

---

## 1. Assignment Details

| Item               | Details                                                        |
| ------------------ | -------------------------------------------------------------- |
| **Course**         | ISEC 2700 – Information Systems Security                       |
| **Workshop Title** | Secure Remote Access Using VPN Technologies                    |
| **Type**           | Hands-On Workshop                                              |
| **Delivery Mode**  | In-Class / Guided Lab                                          |
| **Estimated Time** | 2.5–3 hours                                                    |
| **Weighting**      | Assessment – Practice & Learning Activity (Not Graded)         |
| **Prerequisites**  | Basic networking concepts, TCP/IP, authentication fundamentals |
| **Tools Required** | FortiClient VPN (latest version), Internet access              |
| **Submission**     | Screenshots + reflection                                       |

---

## 2. Overview / Purpose / Objectives

Modern organizations rely heavily on **secure remote access** to internal systems, cloud services, and protected networks. Virtual Private Networks (VPNs) are one of the foundational technologies used to accomplish this securely.

In this workshop, students will:

* Learn **how VPNs work**
* Compare **SSL-VPN vs IPsec VPN**
* Configure a **real enterprise VPN client**
* Connect securely to the **NSCC CCN network**
* Understand **what is happening on the VPN server side**

This lab mirrors **real industry practice**, using the same technology deployed in enterprise environments.

---

## 3. Learning Outcomes Addressed

By the end of this workshop, students will be able to:

* Explain the purpose of VPNs in secure network design
* Distinguish between SSL-VPN and IPsec VPN technologies
* Configure a VPN client using enterprise authentication (SAML / Azure AD)
* Describe how VPNs integrate with identity, encryption, and access control
* Analyze VPNs within a layered security architecture

---

## 4. Assignment Description / Use Case

### Scenario

You are a student accessing internal NSCC systems remotely.

These systems are **not publicly accessible** and must be protected from:

* Unauthorized access
* Credential theft
* Network eavesdropping
* Public Wi-Fi attacks

To safely connect, NSCC uses an **enterprise SSL-VPN solution** backed by:

* Encrypted tunnels
* Central authentication (Azure Active Directory)
* Role-based access control
* Logging and monitoring

You will configure **FortiClient VPN**, the same client used by many organizations worldwide.

---

## 5. Background Theory

---

## 5.1 What Is a VPN?

A **Virtual Private Network (VPN)** creates a **secure, encrypted tunnel** across an untrusted network (the Internet).

From a security perspective, a VPN provides:

* 🔒 **Confidentiality** – encryption protects data
* ✅ **Integrity** – traffic cannot be altered
* 👤 **Authentication** – users must prove identity
* 🧭 **Access control** – only approved resources are reachable

When connected, your computer behaves as if it were **logically inside the private network**.

---

## 5.2 Why VPNs Exist

Without a VPN:

* Traffic travels in clear routing paths
* Attackers can observe metadata
* Internal systems must be exposed publicly

With a VPN:

* Internal IP ranges remain private
* Access is identity-based
* Traffic is encrypted end-to-end

VPNs are a cornerstone of:

* Remote work
* Hybrid learning
* Secure administration
* Zero Trust architectures

---

## 5.3 SSL-VPN vs IPsec VPN

| Feature             | SSL-VPN                 | IPsec VPN               |
| ------------------- | ----------------------- | ----------------------- |
| OSI Layer           | Application / Transport | Network                 |
| Protocols           | TLS / HTTPS             | ESP, IKE                |
| Firewall Friendly   | ✅ Very                  | ⚠️ Sometimes blocked    |
| User Authentication | Strong (SAML, MFA)      | Often certificate-based |
| Common Use          | Remote users            | Site-to-site            |
| Client Experience   | Easy                    | More complex            |

### Key takeaway:

* **SSL-VPN** is ideal for **end users**
* **IPsec** is ideal for **network-to-network tunnels**

NSCC uses **SSL-VPN** because it integrates cleanly with modern identity systems.

---

## 6. VPN Server-Side Architecture (Conceptual)

Students do **not** configure this — but must understand it.

### On the NSCC side:

```
Student Laptop
   |
   | Encrypted TLS Tunnel
   |
Internet
   |
FortiGate Firewall
   |
Authentication (Azure AD / SAML)
   |
Policy Enforcement
   |
Internal CCN Network
```

### Components Explained

#### 1. FortiGate Firewall (VPN Server)

* Terminates SSL-VPN connections
* Handles encryption and tunneling
* Applies firewall policies

#### 2. Identity Provider (Azure Active Directory)

* Validates your NSCC credentials
* Performs SSO authentication
* Issues a SAML token

#### 3. Access Policies

* Determines what network resources you can reach
* Enforces least privilege
* Logs all connections

This design aligns directly with:

* **Defense in Depth**
* **Least Privilege**
* **Separation of Duties**
* **Centralized Identity Management**

---

## 7. Workshop Tasks / Instructions

---

## Task 1 — Download FortiClient VPN

1. Visit the official Fortinet website
2. Download the **latest FortiClient VPN (Free)** version
3. Install using default options

⚠️ Do **not** download cracked or third-party versions.

---

## Task 2 — Create a New VPN Connection

Open **FortiClient VPN** and configure the following:

---

### 🔧 VPN Configuration Settings

#### 1️⃣ VPN Type

* **Use SSL-VPN**

**Explanation:**
SSL-VPN uses TLS encryption (the same technology as HTTPS) and is ideal for remote access through firewalls and NAT.

---

#### 2️⃣ Connection Name

```
NSCC
```

This is simply a label for your connection profile.

---

#### 3️⃣ Description

```
Connectivity to the NSCC CCN Network
```

Descriptions are useful in enterprise environments with multiple VPNs.

---

#### 4️⃣ Remote Gateway

```
https://vpn.nscc.ca:443/ccn
```

**Explanation:**

* `vpn.nscc.ca` = VPN server hostname
* `443` = HTTPS / TLS port
* `/ccn` = VPN realm or portal

This directs authentication to the correct access group.

---

#### 5️⃣ Customize Port

```
443
```

**Why 443?**

* Same port as secure web traffic
* Rarely blocked by firewalls
* Ideal for roaming users and public Wi-Fi

---

#### 6️⃣ Enable Single Sign-On (SSO)

✅ **Enable SSO for VPN tunnel**

**Explanation:**
Allows FortiClient to integrate with browser-based authentication.

---

#### 7️⃣ Use External Browser for SAML Authentication

✅ **Enable**

**Why this matters:**

* Azure AD requires modern browser support
* Supports MFA
* Improves security and compatibility

---

#### 8️⃣ Do NOT Enable Auto-Login with Azure AD

❌ Leave unchecked

**Reason:**
Auto-login bypasses intentional authentication steps and can cause session or token errors.

---

#### 9️⃣ Client Certificate

Select:

```
None
```

**Explanation:**
This VPN uses **identity-based authentication**, not certificate-based authentication.

Certificates are more common with IPsec.

---

#### 🔟 Dual-Stack IPv4 / IPv6

❌ Do **not** enable

**Reason:**

* NSCC VPN operates over IPv4
* Dual-stack can cause routing issues
* IPv6 VPN support varies by enterprise policy

---

## Task 3 — Connect to the VPN

1. Click **Connect**
2. Your browser will open automatically
3. Sign in using your NSCC credentials
4. Complete any MFA prompts
5. Return to FortiClient and verify:

✅ VPN status = **Connected**

---

## Task 4 — Verify VPN Connection

Once connected:

* Observe assigned IP address
* Confirm tunnel is active
* Note connection duration and status

(Optional instructor demonstration: show split tunneling concept.)

---

## 8. Deliverables

Students must submit:

1. Screenshot of FortiClient configuration (sensitive data hidden)
2. Screenshot showing **Connected** status
3. Short reflection response

---

## 9. Reflection Questions

Students must answer the following:

1. Why is SSL-VPN preferred over IPsec for remote users?
2. How does SAML improve VPN security?
3. What security risks would exist without a VPN?
4. How does this VPN implementation support:

   * Confidentiality
   * Integrity
   * Authentication
5. Where does VPN fit within the Security Architecture Model?

---

## 10. Resources / Equipment

* FortiClient VPN (latest)
* Internet access
* NSCC credentials
* Lecture notes on:

  * Encryption
  * Identity and Access Management
  * Network security

---

## 11. Academic Policies

All work must comply with NSCC academic integrity policies.

Do not share credentials or screenshots containing sensitive information.

---

## 12. Copyright Notice

© Nova Scotia Community College
For educational use only.

---

### ✅ Instructor Notes (Optional)

* This workshop maps cleanly to:

  * **IAM**
  * **Network Security**
  * **Secure Remote Access**
* Excellent lead-in to:

  * Zero Trust discussion
  * VPN vs ZTNA
  * Split tunneling risks
  * Endpoint trust models
