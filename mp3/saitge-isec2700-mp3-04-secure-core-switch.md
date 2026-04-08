# **ISEC2700 – Mini Project 3 (MP03-04)**

## **Phase 04: Secure Core Switch (VLAN + Segmentation Enforcement)**

**Course:** ISEC2700 – Intro to Information Security Practices<br>
**Instructor:** Davis Boudreau<br>
**Type:** Mini-Project Phase<br>
**Mode:** Individual<br>
**Estimated Time:** 1–2 hours<br>
**Prerequisite:** MP03-03 completed<br>

---

# **1. Overview / Purpose**

In this phase, you will configure the **Core Switch (Cisco IOSvL2)** to:

* implement **VLAN segmentation**
* enable **inter-VLAN routing using SVIs**
* enforce **strict access control between application tiers**

---

## 🎯 Your Goals

You will:

* configure VLANs 30, 40, and 50
* configure **SVIs (Switch Virtual Interfaces)**
* apply **ACLs to control inter-VLAN traffic**
* restrict communication between:

  * Web tier (VLAN 40)
  * Database tier (VLAN 50)
* validate using:

  * **pgAdmin container**
  * **debian-iptools container**

---

# **2. What You Are Building (Security Model)**

---

## 🔐 Segmentation Model

| VLAN    | Purpose                  | Network         |
| ------- | ------------------------ | --------------- |
| VLAN 30 | App Entry (Proxy → Core) | 192.168.30.0/24 |
| VLAN 40 | Web Tier                 | 192.168.40.0/24 |
| VLAN 50 | Database Tier            | 192.168.50.0/24 |

---

## 🔥 Security Policy

### ✅ Allowed:

| Source            | Destination | Port        |
| ----------------- | ----------- | ----------- |
| VLAN 30 → VLAN 40 | Web         | TCP 80, 443 |
| VLAN 40 → VLAN 50 | Database    | TCP 5432    |

---

### ❌ Denied:

* ALL other inter-VLAN traffic
* Direct DB access from VLAN 30
* Any non-Postgres traffic to VLAN 50

---

## 🧠 Critical Concept

> 🚨 “Just because routing is possible does NOT mean it should be allowed.”

---

# **3. Key Concepts (Teaching Section)**

---

## 🔹 What is an SVI?

An SVI is a virtual interface that:

* represents a VLAN
* allows routing between VLANs

---

### Example:

```cisco
interface vlan 40
 ip address 192.168.40.1 255.255.255.0
```

---

## 🔹 Inter-VLAN Routing

* traffic between VLANs is routed via SVIs
* this is where we enforce ACLs

---

## 🔹 ACL Placement (CRITICAL)

ACLs are applied:

👉 **on the SVI interface**

👉 **inbound direction (best practice)**

---

# **4. Network Reference**

| Device      | IP           |
| ----------- | ------------ |
| Core VLAN30 | 192.168.30.2 |
| Core VLAN40 | 192.168.40.1 |
| Core VLAN50 | 192.168.50.1 |
| Web Server  | 192.168.40.2 |
| DB Server   | 192.168.50.2 |

---

# **5. Step-by-Step Configuration**

---

## 🔧 Step 1 – Access Switch

```cisco
enable
configure terminal
hostname CORE-SW
```

---

## 🔧 Step 2 – Create VLANs

```cisco
vlan 30
 name APP_NET

vlan 40
 name WEB_NET

vlan 50
 name DB_NET
```

---

## 🔧 Step 3 – Assign Ports**

```cisco
interface g0/0
 switchport mode access
 switchport access vlan 30

interface g0/1
 switchport mode access
 switchport access vlan 40

interface g0/2
 switchport mode access
 switchport access vlan 50
```

---

## 🔧 Step 4 – Configure SVIs

```cisco
interface vlan 30
 ip address 192.168.30.2 255.255.255.0
 no shutdown

interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown

interface vlan 50
 ip address 192.168.50.1 255.255.255.0
 no shutdown
```

---

## 🔧 Step 5 – Enable Routing

```cisco
ip routing
```

---

# **6. Configure ACLs (Segmentation Enforcement)**

---

## 🔥 ACL 1 – VLAN 30 → VLAN 40 (Web Access Only)

```cisco
ip access-list extended VLAN30-FILTER

 permit tcp 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255 eq 80
 permit tcp 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255 eq 443

 deny ip any any
```

---

## 🔧 Apply ACL

```cisco
interface vlan 30
 ip access-group VLAN30-FILTER in
```

---

---

## 🔥 ACL 2 – VLAN 40 → VLAN 50 (Postgres Only)

```cisco
ip access-list extended VLAN40-FILTER

 permit tcp 192.168.40.0 0.0.0.255 host 192.168.50.2 eq 5432

 deny ip any any
```

---

## 🔧 Apply ACL

```cisco
interface vlan 40
 ip access-group VLAN40-FILTER in
```

---

# **7. Testing Environment Setup**

---

## 🧪 Containers Used

| Tool           | Purpose              |
| -------------- | -------------------- |
| pgAdmin        | Test DB connectivity |
| debian-iptools | Network testing      |

---

---

# **8. Testing – Debian IP Tools**

---

## 🔧 Test 1 – Web Access

```bash
curl http://192.168.40.2
```

✅ Should work from VLAN 30

---

## 🔧 Test 2 – DB Access (Should FAIL)

```bash
nc -zv 192.168.50.2 5432
```

❌ Should fail from VLAN 30

---

---

# **9. Testing – pgAdmin (Database Access)**

---

## 🔧 Step 1 – Connect pgAdmin container to VLAN 40

---

## 🔧 Step 2 – Configure pgAdmin connection

| Field | Value        |
| ----- | ------------ |
| Host  | 192.168.50.2 |
| Port  | 5432         |

---

## ✅ Expected Result

* Connection SUCCESS from VLAN 40

---

## ❌ Negative Test

From VLAN 30:

* DB access should FAIL

---

# **10. Advanced Testing (Optional)**

---

## 🔍 Port Scan

```bash
nmap 192.168.50.2
```

Expected:

* only port 5432 visible (from VLAN 40)
* none from VLAN 30

---

---

## 🔍 Packet Capture

```bash
tcpdump -i eth0 port 5432
```

Observe allowed vs blocked traffic

---

---

# **11. Verification Commands (Switch)**

```cisco
show vlan brief
show ip interface brief
show access-lists
show running-config
```

---

# **12. Troubleshooting**

---

## ❌ No routing

* forgot `ip routing`

---

## ❌ All traffic blocked

* ACL too restrictive
* rule order incorrect

---

## ❌ DB unreachable from VLAN 40

* wrong ACL destination
* wrong IP

---

## ❌ Web unreachable

* wrong VLAN assignment

---

# **13. Deliverables**

* VLAN configuration screenshot
* ACL configuration screenshot
* pgAdmin successful connection
* failed connection test evidence
* short explanation:

Explain:

* VLAN segmentation
* why DB is restricted
* how ACL enforces security

---

# **14. Reflection Questions**

1. Why should the database not be directly accessible from VLAN 30?
2. Why is only port 5432 allowed between web and DB?
3. What would happen if we removed the ACL?
4. Why are ACLs applied inbound on SVIs?
5. How does this improve security compared to flat networks?

---

# **15. Assessment (Suggested)**

| Criteria           | Marks |
| ------------------ | ----- |
| VLAN Setup         | 5     |
| SVI Config         | 5     |
| ACL Implementation | 10    |
| Testing            | 5     |
| Documentation      | 5     |

**Total: 30**

---

# **16. Instructor Notes (Critical Teaching Moment)**

This phase teaches:

* **east-west traffic control**
* **zero trust principles inside the network**
* **segmentation enforcement**

---

## Common Student Mistakes

* applying ACL to wrong interface
* using wrong subnet in ACL
* forgetting implicit deny
* misunderstanding traffic direction

---
