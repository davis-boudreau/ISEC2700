# **ISEC2700 – Mini Project 3 (MP03-01)**

## **Phase 01: Secure ISP Router (NAT + ACLs + Controlled Inbound Access)**

**Course:** ISEC2700 – Intro to Information Security Practices
**Instructor:** Davis Boudreau
**Type:** Mini-Project Phase
**Mode:** Individual
**Estimated Time:** 1–3 hours
**Prerequisite:** MP03-00 completed

---

# **1. Overview / Purpose**

In this phase, you will configure the **ISP Router (ISP-RTR)** to act as the **secure boundary** between:

* the external ISP network (`172.16.184.0/24`)
* the internal edge network (`192.168.10.0/24`)

This router will:

* enable outbound Internet access using **NAT (PAT)**
* allow controlled inbound web access (**ports 80/443 only**) to the DMZ proxy
* enforce traffic rules using **Access Control Lists (ACLs)**

---

# **2. What You Are Building (Big Picture)**

### ✅ Allowed Traffic

* Internal → Internet (via NAT)
* Internet → Reverse Proxy (HTTP/HTTPS only)

### ❌ Blocked Traffic

* Direct inbound access to internal networks
* Any non-approved services

---

# **3. Key Concepts (Must Understand Before Configuring)**

---

## 🔹 NAT (Network Address Translation)

Private IP addresses (like `192.168.x.x`) cannot be routed on external networks.

NAT solves this problem.

---

### Two NAT types used in this lab:

| Type                         | Purpose                                   |
| ---------------------------- | ----------------------------------------- |
| PAT (Overload)               | Many internal devices share 1 external IP |
| Static NAT (Port Forwarding) | External users reach an internal service  |

---

### 🧠 Example Flow (Outbound)

```
192.168.10.3 → 8.8.8.8
↓
Translated to:
172.16.184.2XX → 8.8.8.8
```

---

### 🧠 Example Flow (Inbound Web Traffic)

```
Internet → 172.16.184.2XX:80
↓
Translated to:
192.168.20.2:80 (DMZ Proxy)
```

---

## 🔹 ACLs (Access Control Lists)

ACLs filter traffic.

---

### How ACLs work:

1. Evaluated **top-down**
2. First match is applied
3. Implicit:

```text
deny all
```

---

### Two Types of ACLs

---

### ✅ Standard ACL

* Matches **source IP only**
* Used for:

  * NAT identification

Example:

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
```

---

### ✅ Extended ACL

* Matches:

  * source IP
  * destination IP
  * protocol
  * ports

Used for:

* **security filtering**

---

### 🔹 Wildcard Mask Reminder

| Subnet Mask     | Wildcard  |
| --------------- | --------- |
| 255.255.255.0   | 0.0.0.255 |
| 255.255.255.255 | 0.0.0.0   |

---

## 🔹 CRITICAL: NAT vs ACL (Students often confuse this)

| Function     | Purpose                         |
| ------------ | ------------------------------- |
| NAT ACL      | identifies traffic to translate |
| Security ACL | allows/blocks traffic           |

👉 These are **NOT the same thing**

---

## 🔹 Security Principle: Least Privilege

> Only allow what is required.

We ONLY expose:

* HTTP (80)
* HTTPS (443)

Everything else is blocked.

---

# **4. Network Reference**

| Component   | Address        |
| ----------- | -------------- |
| Router G0/0 | 172.16.184.2XX |
| Router G0/1 | 192.168.10.1   |
| Gateway     | 172.16.184.250 |
| DMZ Proxy   | 192.168.20.2   |
| Test Host   | 192.168.10.3   |

---

# **5. Router Role in This Phase**

The router performs:

1. Routing
2. NAT (inside → outside)
3. Port forwarding (outside → DMZ)
4. Traffic filtering (ACL)

---

# **6. Step-by-Step Configuration**

---

## **Step 1 – Access Router**

```cisco
enable
configure terminal
```

---

## **Step 2 – Basic Setup**

```cisco
hostname ISP-RTR
no ip domain-lookup
service password-encryption
banner motd #Authorized access only#
```

---

## **Step 3 – Configure Interfaces**

```cisco
interface g0/0
 description OUTSIDE_TO_ISP
 ip address 172.16.184.2XX 255.255.255.0
 no shutdown

interface g0/1
 description INSIDE_TO_EDGE
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

---

## **Step 4 – Default Route**

```cisco
ip route 0.0.0.0 0.0.0.0 172.16.184.250
```

---

## **Step 5 – NAT Interface Roles**

```cisco
interface g0/0
 ip nat outside

interface g0/1
 ip nat inside
```

---

# **7. Configure NAT (Outbound Access)**

---

## **Step 6 – NAT ACL**

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
```

---

## **Step 7 – Enable PAT**

```cisco
ip nat inside source list 1 interface g0/0 overload
```

---

# **8. Configure Port Forwarding (Inbound Access)**

---

## **Step 8 – Static NAT**

```cisco
ip nat inside source static tcp 192.168.20.2 80 interface g0/0 80
ip nat inside source static tcp 192.168.20.2 443 interface g0/0 443
```

---

# **9. Configure Security ACL**

---

## **Step 9 – Create ACL**

```cisco
ip access-list extended OUTSIDE-IN

 permit tcp any host 172.16.184.2XX eq 80
 permit tcp any host 172.16.184.2XX eq 443

 permit tcp any any established

 permit icmp any any echo-reply
 permit icmp any any time-exceeded
 permit icmp any any unreachable

 deny ip any 192.168.10.0 0.0.0.255

 permit ip any any
```

---

## 🔥 Explanation (Students MUST understand this)

| Rule           | Purpose                     |
| -------------- | --------------------------- |
| 80/443 permit  | Allow public web access     |
| established    | Allow return traffic        |
| ICMP           | Enable troubleshooting      |
| deny internal  | Block direct access         |
| permit any any | Prevent accidental breakage |

---

## **Step 10 – Apply ACL**

```cisco
interface g0/0
 ip access-group OUTSIDE-IN in
```

---

# **10. Configure Test Host (Switch2)**

---

## **Step 11 – Connect Container**

* Use: `debian-iptools`
* Connect to Switch2

---

## **Step 12 – Configure IP**

```bash
ip addr flush dev eth0
ip addr add 192.168.10.3/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.10.1
```

---

# **11. Validation**

---

## ✅ Test 1 – Router Reachability

```bash
ping 192.168.10.1
```

---

## ✅ Test 2 – Internet (NAT)

```bash
ping 8.8.8.8
```

---

## ✅ Test 3 – NAT Table

```cisco
show ip nat translations
```

---

## ✅ Test 4 – Web Access

From Chromium:

```
http://172.16.184.2XX
```

---

# **12. Verification Commands**

```cisco
show ip interface brief
show ip route
show ip nat translations
show ip nat statistics
show access-lists
show running-config
```

---

# **13. Troubleshooting Guide**

---

## ❌ No Internet

* NAT missing
* default route wrong
* wrong gateway on test host

---

## ❌ No NAT entries

* ACL mismatch
* wrong interface roles

---

## ❌ ACL blocking traffic

* rule order incorrect
* missing permit

---

## ❌ Cannot reach DMZ

* port forwarding missing
* ACL not permitting 80/443

---

# **14. Deliverables**

* Router config screenshot
* NAT table screenshot
* Test host config
* Successful ping
* Web test
* Short explanation:

Explain:

* NAT vs ACL
* why only 80/443 are open

---

# **15. Reflection Questions**

1. Why is NAT required?
2. Difference between PAT and static NAT?
3. Why restrict inbound ports?
4. What happens if ACL removed?
5. Why is rule order critical?

---

# **16. Assessment (Suggested)**

| Criteria         | Marks |
| ---------------- | ----- |
| Interface Config | 5     |
| Routing          | 5     |
| NAT              | 5     |
| Port Forwarding  | 5     |
| ACL Security     | 5     |
| Validation       | 5     |

**Total: 30**

---
