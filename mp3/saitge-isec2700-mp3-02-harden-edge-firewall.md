# **ISEC2700 – Mini Project 3 (MP03-02)**

## **Phase 02: Harden Edge Firewall (pfSense 2.8.1)**

**Course:** ISEC2700 – Intro to Information Security Practices<br>
**Instructor:** Davis Boudreau<br>
**Type:** Mini-Project Phase<br>
**Mode:** Individual<br>
**Estimated Time:** 1–3 hours<br>
**Prerequisite:** MP03-01 completed<br>

---

# **1. Overview / Purpose**

In this phase, you will configure and harden the **Edge Firewall (pfSense 2.8.1)**.

This firewall sits between:

* **Router (untrusted edge)** → `192.168.10.0/24`
* **DMZ network** → `192.168.20.0/24`

---

## 🎯 Your Goals

You will:

* access and configure the **pfSense Web Admin GUI**
* define **firewall rules**
* implement **default deny + explicit allow**
* control traffic between:

  * edge network
  * DMZ
* validate access using the **Chromium test browser**

---

# **2. What You Are Building (Security Model)**

---

## 🔐 Trust Zones

| Zone       | Network         | Trust Level  |
| ---------- | --------------- | ------------ |
| WAN (Edge) | 192.168.10.0/24 | Untrusted    |
| DMZ        | 192.168.20.0/24 | Semi-trusted |

---

## 🔥 Firewall Role

The firewall:

* **blocks everything by default**
* only allows:

  * required traffic
  * explicitly defined rules

---

## 🧠 Key Principle

> 🚨 “If you didn’t explicitly allow it — it is blocked.”

---

# **3. Firewall Concepts**

---

## 🔹 Stateless vs Stateful Firewalls

| Type      | Behavior                             |
| --------- | ------------------------------------ |
| Stateless | Evaluates each packet individually   |
| Stateful  | Tracks connections (pfSense default) |

---

### ✅ pfSense is STATEFUL

This means:

* if you allow outbound traffic
* return traffic is automatically allowed

---

## 🔹 Rule Processing (CRITICAL)

* Rules are evaluated **top-down**
* First match wins
* Default = **deny all**

---

## 🔹 Interface-Based Rules (VERY IMPORTANT)

pfSense rules are applied:

👉 **on the interface where traffic ENTERS**

---

### Example:

| Traffic   | Rule goes on  |
| --------- | ------------- |
| WAN → DMZ | WAN interface |
| DMZ → WAN | DMZ interface |

---

# **4. Network Reference**

| Interface | Name | IP           |
| --------- | ---- | ------------ |
| em0       | WAN  | 192.168.10.2 |
| em1       | DMZ  | 192.168.20.1 |

---

# **5. Initial Access to pfSense Web GUI**

---

## 🔧 Step 1 – Connect Chromium to Switch2

Ensure Chromium is connected to:

```
Switch2 (192.168.10.0/24)
```

---

## 🔧 Step 2 – Assign DHCP or Static IP

If needed:

* IP: `192.168.10.X`
* Gateway: `192.168.10.1`

---

## 🔧 Step 3 – Open Web Browser

Navigate to:

```
https://192.168.10.2
```

---

## 🔐 Step 4 – Login

Default credentials:

| Field    | Value   |
| -------- | ------- |
| Username | admin   |
| Password | pfsense |

---

⚠️ Accept SSL warning (self-signed certificate)

---

# **6. Initial pfSense Setup (Wizard Overview)**

Students should:

* confirm hostname
* confirm WAN interface
* confirm LAN (DMZ) interface
* skip advanced options for now

---

# **7. Baseline Firewall Behavior**

By default:

* WAN = **block all inbound**
* LAN/DMZ = **allow outbound**

---

## 🧠 Important Insight

Right now:

* DMZ → Internet = allowed
* WAN → DMZ = blocked

---

# **8. Hardening Tasks**

---

# 🔐 **Task 1 – Change Admin Password**

System → User Manager

* change admin password
* document securely

---

# 🔐 **Task 2 – Disable Ping from WAN**

Navigate:

```
Firewall → Rules → WAN
```

Ensure:

* NO rule allows ICMP echo requests

---

# 🔐 **Task 3 – Allow HTTP/HTTPS to DMZ Proxy**

---

## 🔧 Add Rule on WAN Interface

Firewall → Rules → WAN → Add

---

### Rule 1 – HTTP

| Field            | Value        |
| ---------------- | ------------ |
| Action           | Pass         |
| Interface        | WAN          |
| Protocol         | TCP          |
| Source           | any          |
| Destination      | 192.168.20.2 |
| Destination Port | 80           |

---

### Rule 2 – HTTPS

| Field            | Value        |
| ---------------- | ------------ |
| Action           | Pass         |
| Interface        | WAN          |
| Protocol         | TCP          |
| Source           | any          |
| Destination      | 192.168.20.2 |
| Destination Port | 443          |

---

## 🧠 Explain to Students

These rules allow:

* Internet users → Reverse Proxy ONLY
* nothing else exposed

---

# 🔐 **Task 4 – Block Direct Access to Internal Networks**

Add rule:

| Field       | Value           |
| ----------- | --------------- |
| Action      | Block           |
| Source      | any             |
| Destination | 192.168.10.0/24 |

---

---

# 🔐 **Task 5 – Restrict DMZ Outbound Traffic (Basic Hardening)**

Go to:

```
Firewall → Rules → DMZ
```

---

## Allow only required outbound traffic

### Rule 1 – Allow HTTP

* Destination: any
* Port: 80

### Rule 2 – Allow HTTPS

* Destination: any
* Port: 443

---

### Final Rule

Block everything else

---

## 🧠 Teaching Point

> DMZ systems should NOT have unrestricted outbound access

---

# **9. Apply Changes**

Click:

```
Apply Changes
```

---

# **10. Validation Testing**

---

## ✅ Test 1 – From Chromium

Test:

```
http://172.16.184.2XX
```

Expected:

* reaches proxy

---

## ❌ Test 2 – Blocked Access

Try:

```
ssh 172.16.184.2XX
```

Expected:

* blocked

---

## ✅ Test 3 – DMZ Outbound

From proxy (later phase):

* should reach Internet on 80/443 only

---

# **11. Logs and Monitoring**

---

## 🔍 View Logs

```
Status → System Logs → Firewall
```

Students should:

* observe blocked traffic
* observe allowed traffic

---

## 🧠 Teaching Insight

Logs = **evidence of security enforcement**

---

# **12. Troubleshooting**

---

## ❌ Cannot access Web GUI

* wrong network
* wrong IP
* interface down

---

## ❌ HTTP not working

* NAT missing (router issue)
* firewall rule missing
* wrong destination IP

---

## ❌ Everything blocked

* rule order incorrect
* forgot “pass” rule

---

# **13. Deliverables**

* Screenshot of firewall rules (WAN + DMZ)
* Screenshot of Web GUI access
* Screenshot of successful HTTP test
* Screenshot of firewall logs
* Short explanation:

Explain:

* stateful firewall
* why only ports 80/443 allowed
* difference between router ACL and firewall rules

---

# **14. Reflection Questions**

1. Why is pfSense considered stateful?
2. Why are rules applied on the interface where traffic enters?
3. Why is the DMZ not fully trusted?
4. What would happen if we allowed all outbound traffic from DMZ?
5. How does this firewall improve security compared to router ACLs?

---

# **15. Assessment (Suggested)**

| Criteria           | Marks |
| ------------------ | ----- |
| Web GUI Access     | 5     |
| Rule Configuration | 10    |
| Security Hardening | 5     |
| Testing            | 5     |
| Documentation      | 5     |

**Total: 30**

---

# **16. Instructor Notes**

This is where students shift from:

* “network works” → “network is controlled”

Common confusion:

* rules direction (inbound vs outbound)
* interface selection
* NAT vs firewall responsibilities

---
