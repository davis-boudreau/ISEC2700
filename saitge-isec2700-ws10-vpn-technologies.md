# 🔐 Virtual Private Networks (VPNs)

**Comprehensive Student Tutorial & Assignment**

---

## 1. Assignment Details

| Item            | Description                                         |
| --------------- | --------------------------------------------------- |
| Course          | Networking & Security / Information Security        |
| Topic           | VPN Types, Architectures, and Secure Communications |
| Type            | Tutorial + Applied Analysis                         |
| Estimated Time  | 3–4 hours                                           |
| Assessment Type | Knowledge + Design Thinking                         |
| Weight          | Instructor defined                                  |

---

## 2. Overview / Purpose / Objectives

VPNs are not just “remote login tools.”

In real organizations, VPNs are used for:

* Secure site-to-site communication
* Remote user access
* Secure application publishing
* Server-to-server encryption
* Email system protection

This tutorial introduces VPNs using **architectural thinking**, not just protocol memorization.

Students will learn:

* Different **types of VPNs**
* How tunnels are created and controlled
* When to use **route-based vs policy-based VPNs**
* Differences between **client-based and clientless VPNs**
* How encryption is used for **email server communications**

---

## 3. Learning Outcomes Addressed

Students will be able to:

* Explain major VPN architectures
* Differentiate site-to-site and remote-access VPNs
* Compare route-based and policy-based tunnels
* Identify secure protocols for different scenarios
* Apply VPN concepts to real enterprise use cases

---

## 4. VPN Architecture Overview

At a high level, VPNs fall into **three major categories**:

1. **Site-to-Site VPNs**
2. **Remote Access VPNs**
3. **Server-to-Server Secure Communications**

Each serves a different security purpose.

---

# 🧱 PART 1 — SITE-TO-SITE VPNs

Site-to-site VPNs permanently connect **two networks together**.

Users do not log in — **networks trust networks**.

### Example

* Head office ↔ Branch office
* Data center ↔ Cloud network
* Campus ↔ Campus

```
LAN A ──[Firewall/VPN]====Encrypted Tunnel====[Firewall/VPN]── LAN B
```

---

## 1️⃣ Policy-Based VPN

### How it works

* Traffic is encrypted only if it matches specific rules
* Uses firewall policies to define:

  * Source
  * Destination
  * Protocol

### Example policy

```
If traffic is:
10.0.1.0/24 → 10.0.2.0/24
Then encrypt using IPsec
```

### Characteristics

✅ Simple
✅ Easy to configure
❌ Limited scalability
❌ Not flexible for dynamic routing

### Common use

* Small networks
* Basic site-to-site tunnels

---

## 2️⃣ Route-Based VPN

### How it works

* Creates a **virtual tunnel interface (VTI)**
* VPN behaves like a network interface
* Routing tables decide what goes through the tunnel

### Example

```
ip route 10.0.2.0/24 via vpn-tunnel1
```

### Characteristics

✅ Supports dynamic routing (OSPF, BGP)
✅ Scales well
✅ Cloud-friendly
✅ Enterprise standard

### Common use

* Large organizations
* Multi-site networks
* Cloud VPNs (AWS, Azure)

### Industry reality

🟢 **Route-based VPNs are now the standard**

---

## Protocols Used in Site-to-Site VPNs

| Protocol | Use                       |
| -------- | ------------------------- |
| IPsec    | Primary encryption method |
| IKEv2    | Key exchange              |
| AES      | Data encryption           |
| SHA-2    | Integrity                 |

---

# 🧑‍💻 PART 2 — REMOTE ACCESS VPNs

Remote access VPNs allow **individual users** to connect securely to an internal network.

Unlike site-to-site VPNs:

* Authentication is identity-based
* MFA is commonly required

---

## Remote Access VPN Models

### 1. Client-Based VPN

### 2. Clientless VPN (Browser-based)

---

## 1️⃣ Client-Based Remote Access VPN

Requires software installed on the device.

### Examples

* FortiClient
* Cisco AnyConnect
* GlobalProtect

---

### A. IPsec Client VPN

**Uses:**

* IPsec + IKEv2

**Strengths**
✅ Strong encryption
✅ Efficient
✅ Suitable for corporate devices

**Weaknesses**
⚠️ Firewall traversal issues
⚠️ More complex configuration

**Common use**

* Company laptops
* Always-on VPN

---

### B. TLS / SSL Client VPN

**Uses:**

* TLS (same as HTTPS)
* TCP 443

**Strengths**
✅ Works through almost all firewalls
✅ Excellent for students and remote users
✅ Supports MFA and SSO

**Common use**

* FortiClient SSL-VPN
* Hybrid learning environments
* BYOD devices

🟢 **Most common modern remote access VPN**

---

## 2️⃣ Clientless VPN (Web Portal)

No client software required.

### How it works

* User logs in via web browser
* Firewall acts as a secure proxy
* Access limited to:

  * Web apps
  * File shares
  * Internal portals

### Example

```
https://vpn.organization.ca
```

### Strengths

✅ No installation required
✅ Ideal for shared computers
❌ Limited application support

### Common use

* Contractors
* Temporary access
* Emergency access

---

# 📧 PART 3 — SERVER-TO-SERVER SECURE COMMUNICATIONS

Not all encryption uses VPN clients.

Some systems use **application-level encryption**.

---

## Email Server to Server Communications

Email servers communicate using **SMTP**, but modern SMTP is encrypted.

### Technologies used

| Technology   | Purpose                      |
| ------------ | ---------------------------- |
| STARTTLS     | Encrypts SMTP traffic        |
| TLS          | Secures email in transit     |
| Certificates | Server identity verification |

### Example

```
mail.nscc.ca → mail.microsoft.com
Encrypted using TLS
```

### Important distinction

This is **NOT a VPN tunnel**.

Instead:

* Encryption happens at the application layer
* Each message is protected during transit

### Why this matters

* Email crosses many networks
* VPN tunnels are impractical for global mail flow
* TLS provides scalable trust using certificates

---

# 🔐 VPN vs TLS Encryption Comparison

| Use Case              | Technology        |
| --------------------- | ----------------- |
| Site-to-site networks | IPsec VPN         |
| Remote users          | SSL/TLS VPN       |
| Cloud routing         | Route-based IPsec |
| Email delivery        | SMTP over TLS     |
| Web traffic           | HTTPS (TLS)       |

---

# 📊 Summary Architecture Table

| Scenario          | VPN Type         | Tunnel Model      | Protocol |
| ----------------- | ---------------- | ----------------- | -------- |
| Branch ↔ HQ       | Site-to-site     | Route-based       | IPsec    |
| Small office      | Site-to-site     | Policy-based      | IPsec    |
| Remote staff      | Client-based     | SSL/TLS           | TLS 443  |
| Corporate laptops | Client-based     | IPsec             | IKEv2    |
| Contractors       | Clientless       | Browser TLS       | HTTPS    |
| Email servers     | Server-to-server | Application layer | TLS      |

---

# 🧠 Security Architecture Mapping

| Domain      | VPN Role              |
| ----------- | --------------------- |
| Identity    | Authentication, MFA   |
| Network     | Encrypted tunnels     |
| Application | Secure access paths   |
| Data        | Encryption in transit |
| Detection   | VPN logs              |
| Response    | Account revocation    |

---

## 7. Tasks / Instructions

### Task 1 – Knowledge Questions

1. Explain the difference between site-to-site and remote access VPNs.
2. Compare route-based and policy-based VPNs.
3. Why is SSL-VPN widely used for students and remote workers?
4. Why is email secured using TLS instead of VPN tunnels?

---

### Task 2 – Design Scenario

Design secure connectivity for:

* Head office
* Cloud environment
* Remote students
* IT administrators
* Email system

For each:

* VPN type
* Protocol
* Authentication method
* Security justification

---

## 8. Deliverables

Submit a short technical explanation (1–2 pages) including:

* Architecture diagrams (optional)
* Protocol selection
* Security reasoning

---

## 9. Reflection Questions

1. Why is identity as important as encryption?
2. Where does VPN technology fit in Zero Trust?
3. What risks exist if VPN access is not segmented?
4. How does TLS differ from network-level VPN encryption?

---

## 10. Resources / Equipment

* FortiClient VPN
* Firewall diagrams
* Course notes
* Instructor examples

---

## 11. Academic Policies

Follow all academic integrity policies.

---

## 12. Copyright Notice

© Nova Scotia Community College
Educational use only.

---
