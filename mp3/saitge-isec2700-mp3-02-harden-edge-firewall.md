# **ISEC2700 – Mini Project 3 (MP03-02)**

## **Phase 02: Harden Edge Firewall (pfSense 2.8.1)**

**Course:** ISEC2700 – Intro to Information Security Practices
**Instructor:** Davis Boudreau
**Type:** Mini-Project Phase
**Mode:** Individual
**Estimated Time:** 4–6 hours
**Prerequisite:** MP03-01 completed

---

# **1. Overview / Purpose**

In this phase, you will configure and harden the **Edge Firewall (pfSense 2.8.1)**.

This firewall sits between:

* **Router (untrusted WAN)** → `192.168.10.0/24`
* **DMZ network (trusted LAN)** → `192.168.20.0/24`

---

## 🎯 Your Goals

You will:

* access the **pfSense Web Admin GUI (LAN side only)**
* understand **stateful firewall behavior**
* configure **WAN firewall rules**
* implement **default deny + explicit allow**
* correctly configure pfSense for a **lab environment**
* validate traffic using test clients

---

# **2. What You Are Building (Security Model)**

---

## 🔐 Trust Zones

| Zone            | Network         | Trust Level     |
| --------------- | --------------- | --------------- |
| WAN (Untrusted) | 192.168.10.0/24 | External / Edge |
| LAN (DMZ)       | 192.168.20.0/24 | Semi-trusted    |

---

## 🔥 Firewall Role

The firewall:

* blocks all traffic by default
* allows only explicitly permitted traffic
* tracks connections (stateful inspection)

---

## 🧠 Key Principle

> 🚨 “If you didn’t explicitly allow it — it is blocked.”

---

# **3. Critical Lab Configuration (MUST DO FIRST)**

---

## ⚠️ **IMPORTANT: pfSense Default WAN Protections**

By default, pfSense enables:

```text
[✓] Block private networks (RFC1918)
[✓] Block bogon networks
```

---

## 🚨 **Why This Breaks Your Lab**

Your lab uses private IP ranges:

* `192.168.x.x`
* `172.16.x.x`

pfSense assumes:

> “Private IPs on WAN = invalid or spoofed”

👉 Result: **ALL traffic is dropped**, even if rules are correct

---

## 🔧 **Step 1 – Disable These Settings**

Navigate:

```
Interfaces → WAN
```

Uncheck:

```
[ ] Block private networks
[ ] Block bogon networks
```

Click:

* **Save**
* **Apply Changes**

---

## 🧠 Teaching Insight

| Environment            | Setting  |
| ---------------------- | -------- |
| Real Internet firewall | ENABLED  |
| GNS3 lab environment   | DISABLED |

---

## 🧪 Quick Validation

From a test host on WAN (optional later):

```bash
ping 192.168.20.1
```

Expected:

* reachable (once rules are in place)

---

# **4. Firewall Concepts (Teaching Section)**

---

## 🔹 Stateful Firewall

pfSense tracks connections:

* outbound allowed → return traffic allowed automatically

---

## 🔹 Rule Processing

* top-down evaluation
* first match wins
* implicit deny at the end

---

## 🔹 Interface-Based Rules

Rules apply where traffic **enters**:

| Traffic   | Rule Location |
| --------- | ------------- |
| WAN → LAN | WAN interface |
| LAN → WAN | LAN interface |

---

# **5. Network Reference**

| Interface | Name      | IP           |
| --------- | --------- | ------------ |
| em0       | WAN       | 192.168.10.2 |
| em1       | LAN (DMZ) | 192.168.20.1 |

---

# **6. Access pfSense Web GUI (LAN Only)**

---

## 🧠 **Important Concept**

> 🔐 pfSense can only be managed from the **LAN (trusted side)**
> ❌ WAN access is blocked by design

---

## 🔧 Step 2 – Connect Chromium to LAN

Connect Chromium container to:

```
Switch3 (192.168.20.0/24)
```

---

## 🔧 Step 3 – Configure IP (if needed)

```bash
ip addr flush dev eth0
ip addr add 192.168.20.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.20.1
```

---

## 🔧 Step 4 – Access Web GUI

Open browser:

```
http://192.168.20.1
```

---

## 🔐 Step 5 – Login

| Field    | Value   |
| -------- | ------- |
| Username | admin   |
| Password | pfsense |

---

## ⚠️ Important

You will NOT be able to access pfSense from:

* WAN network (192.168.10.0/24)
* external networks

👉 This is intentional and correct

---

# **7. Initial Hardening Tasks**

---

## 🔐 Task 1 – Change Admin Password

Navigate:

```
System → User Manager
```

---

## 🔐 Task 2 – Review Default WAN Rules

Navigate:

```
Firewall → Rules → WAN
```

Ensure:

* no overly permissive rules exist

---

# **8. Configure Required WAN Firewall Rules**

---

## 🔥 Task 3 – Allow HTTP/HTTPS to Reverse Proxy

---

Navigate:

```
Firewall → Rules → WAN → Add
```

---

### Rule 1 – HTTP

| Field       | Value        |
| ----------- | ------------ |
| Action      | Pass         |
| Protocol    | TCP          |
| Source      | any          |
| Destination | 192.168.20.2 |
| Port        | 80           |

---

### Rule 2 – HTTPS

| Field       | Value        |
| ----------- | ------------ |
| Action      | Pass         |
| Protocol    | TCP          |
| Source      | any          |
| Destination | 192.168.20.2 |
| Port        | 443          |

---

## 🧠 Explanation

Allows:

* external users → reverse proxy only

---

# **9. Block Direct Access to Internal Networks**

---

## 🔥 Task 4 – Add Block Rule

| Field       | Value           |
| ----------- | --------------- |
| Action      | Block           |
| Source      | any             |
| Destination | 192.168.10.0/24 |

---

## 🧠 Purpose

Prevents:

* bypassing firewall layers
* direct access to edge/internal systems

---

# **10. Restrict LAN (DMZ) Outbound Traffic**

---

## 🔥 Task 5 – Configure LAN Rules

Navigate:

```
Firewall → Rules → LAN
```

---

### Allow HTTP Out

* Port 80

### Allow HTTPS Out

* Port 443

---

### Final Rule

* Block all other outbound traffic

---

## 🧠 Teaching Point

> DMZ systems should not have unrestricted outbound access

---

# **11. Apply Changes**

Click:

```
Apply Changes
```

---

# **12. Validation Testing**

---

## 🧪 Test Setup

Use two perspectives:

| Client             | Network      | Purpose      |
| ------------------ | ------------ | ------------ |
| Chromium (LAN)     | 192.168.20.x | Admin access |
| External test host | WAN side     | User access  |

---

## ✅ Test 1 – External Access

From WAN-side test client:

```
http://172.16.184.2XX
```

Expected:

* reaches reverse proxy

---

## ❌ Test 2 – Unauthorized Access

```bash
nc -zv 172.16.184.2XX 22
```

Expected:

* blocked

---

## ✅ Test 3 – Logs

Navigate:

```
Status → System Logs → Firewall
```

Observe:

* allowed HTTP/HTTPS
* blocked traffic

---

# **13. Verification Checklist**

Students must confirm:

* WAN private/bogon blocking disabled
* WebGUI accessible from LAN only
* HTTP/HTTPS allowed
* no direct internal access
* outbound restricted from DMZ
* logs show enforcement

---

# **14. Troubleshooting**

---

## ❌ Nothing works

👉 FIRST CHECK:

* “Block private networks” disabled

---

## ❌ Cannot access Web GUI

* not on LAN
* wrong IP
* interface down

---

## ❌ HTTP not working

* NAT issue (router)
* missing WAN rule
* wrong destination

---

## ❌ Everything blocked

* rule order incorrect
* missing allow rules

---

# **15. Deliverables**

Students must submit:

* Screenshot of WAN interface settings
* Screenshot showing private/bogon disabled
* Screenshot of WAN rules
* Screenshot of LAN rules
* Screenshot of WebGUI access
* Screenshot of successful HTTP test
* Screenshot of firewall logs

---

# **16. Reflection Questions**

1. Why can pfSense only be accessed from the LAN side?
2. Why must “block private networks” be disabled in this lab?
3. Why should it remain enabled in real environments?
4. What is the difference between router ACL and firewall rules?
5. Why is only HTTP/HTTPS allowed inbound?
6. What risk exists if DMZ outbound is unrestricted?

---

# **17. Assessment (Suggested)**

| Criteria                   | Marks |
| -------------------------- | ----- |
| WAN Config (critical step) | 5     |
| Rule Configuration         | 10    |
| Security Hardening         | 5     |
| Testing                    | 5     |
| Documentation              | 5     |

**Total: 30**

---

# **18. Instructor Notes**

This phase introduces:

* stateful firewall logic
* interface-based rule design
* real-world firewall management practices

---

## ⚠️ Common Student Mistakes

* forgetting to disable private network blocking
* trying to access GUI from WAN
* applying rules to wrong interface
* misunderstanding rule order

---
