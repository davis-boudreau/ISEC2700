# **ISEC2700 – Mini Project 3 (MP03-01)**

## **Phase 01: Configure & Secure ISP Router (NAT + ACLs)**

**Course:** ISEC2700 – Intro to Information Security Practices
**Instructor:** Davis Boudreau
**Type:** Mini-Project Phase
**Mode:** Individual
**Estimated Time:** 4–6 hours
**Prerequisite:** MP03-00 (Topology Built & Connectivity Verified)

---

# **1. Overview / Purpose**

In this phase, you will configure the **ISP Router (Cisco IOSv)** to act as the **network edge device**.

This router is responsible for:

* connecting your lab to the simulated Internet
* performing **Network Address Translation (NAT)**
* controlling inbound access using **Access Control Lists (ACLs)**

---

## 🎯 Your Goals

You will:

* configure router interfaces
* implement **PAT (Port Address Translation)** for outbound traffic
* configure **static NAT (port forwarding)** for inbound HTTP/HTTPS
* apply **ACLs to secure inbound traffic**
* configure routing (including internal network awareness)
* validate Internet connectivity

---

# **2. Network Role of the ISP Router**

---

## 🌐 Interfaces

| Interface | Role                  | Network         |
| --------- | --------------------- | --------------- |
| g0/0      | Outside (Internet)    | 172.16.184.0/24 |
| g0/1      | Inside (Edge Network) | 192.168.10.0/24 |

---

## 🔐 Key Responsibility

The router acts as:

* 🌍 **Public-facing entry point**
* 🔄 **NAT translator**
* 🚧 **first line of defense (ACLs)**

---

# **3. Addressing Scheme**

---

## 🧾 Router IP Assignment

| Interface | Example (Desk 121) |
| --------- | ------------------ |
| g0/0      | 172.16.184.221     |
| g0/1      | 192.168.10.1       |

---

## 🧠 Pattern

```text
172.16.184.2XX
```

Where `XX` = last two digits of desk number

---

# **4. Step-by-Step Configuration**

---

## 🔧 Step 1 – Access Router

```cisco
enable
configure terminal
hostname ISP-RTR
no ip domain-lookup
service password-encryption
banner motd #Authorized Access Only#
```

---

## 🔧 Step 2 – Configure Interfaces

---

### 🌐 Outside Interface

```cisco
interface g0/0
 description OUTSIDE_TO_ISP
 ip address 172.16.184.2XX 255.255.255.0
 ip nat outside
 no shutdown
```

---

### 🏠 Inside Interface

```cisco
interface g0/1
 description INSIDE_TO_EDGE
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 no shutdown
```

---

# **5. Configure Routing**

---

## 🔥 Step 3 – Default Route (Internet Access)

```cisco
ip route 0.0.0.0 0.0.0.0 172.16.184.250
```

---

## 🔥 Step 4 – DMZ Route (CRITICAL)

```cisco
ip route 192.168.20.0 255.255.255.0 192.168.10.2
```

---

## 🧠 Why This Matters

The **reverse proxy** lives on:

```text
192.168.20.2 (DMZ network)
```

The router must know:

> “To reach 192.168.20.0/24, send traffic to pfSense (192.168.10.2)”

Without this route:

* inbound NAT may fail
* return traffic may be dropped
* connectivity becomes inconsistent

---

# **6. Configure NAT**

---

## 🔹 Step 5 – PAT (Outbound Internet Access)

---

### Define inside network

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
```

---

### Enable PAT

```cisco
ip nat inside source list 1 interface g0/0 overload
```

---

## 🔹 Step 6 – Static NAT (Port Forwarding)

---

### Forward HTTP → Reverse Proxy

```cisco
ip nat inside source static tcp 192.168.20.2 80 interface g0/0 80
```

---

### Forward HTTPS → Reverse Proxy

```cisco
ip nat inside source static tcp 192.168.20.2 443 interface g0/0 443
```

---

## 🧠 Teaching Point

This ensures:

```text
Internet → Router → Reverse Proxy (NOT directly to Web App)
```

---

# **7. Configure Security (ACLs)**

---

## 🔥 Step 7 – Create Inbound ACL

```cisco
ip access-list extended OUTSIDE-IN
```

---

### Allow HTTP

```cisco
permit tcp any host 172.16.184.2XX eq 80
```

---

### Allow HTTPS

```cisco
permit tcp any host 172.16.184.2XX eq 443
```

---

### Allow Return Traffic

```cisco
permit tcp any any established
```

---

### Allow Essential ICMP

```cisco
permit icmp any any echo-reply
permit icmp any any time-exceeded
permit icmp any any unreachable
```

---

### Block Internal Network Access

```cisco
deny ip any 192.168.10.0 0.0.0.255
```

---

### Permit Remaining Traffic

```cisco
permit ip any any
```

---

## 🔧 Apply ACL

```cisco
interface g0/0
 ip access-group OUTSIDE-IN in
```

---

## 🧠 Teaching Insight

> NAT controls **where traffic goes**
> ACL controls **whether it is allowed**

---

# **8. Save Configuration**

```cisco
end
copy running-config startup-config
```

---

# **9. Validation Testing**

---

## 🧪 Test 1 – Internet Connectivity (PAT)

From a device on 192.168.10.x:

```bash
ping 8.8.8.8
```

Expected:

* ✅ success

---

## 🧪 Test 2 – NAT Translation Table

```cisco
show ip nat translations
```

Expected:

* dynamic entries appear

---

## 🧪 Test 3 – External Access

From external network:

```bash
curl http://172.16.184.2XX
```

Expected:

* forwarded to reverse proxy

---

## 🧪 Test 4 – Blocked Ports

```bash
nc -zv 172.16.184.2XX 22
```

Expected:

* ❌ blocked

---

# **10. Verification Commands**

```cisco
show ip interface brief
show ip route
show ip nat translations
show access-lists
```

---

# **11. Troubleshooting**

---

## ❌ Cannot reach DMZ

* missing route:

```cisco
ip route 192.168.20.0 255.255.255.0 192.168.10.2
```

---

## ❌ No Internet

* missing default route
* NAT not configured

---

## ❌ Web not accessible

* NAT pointing to wrong IP
* pfSense rules blocking
* reverse proxy not running

---

## ❌ Everything blocked

* ACL misconfigured
* ACL applied incorrectly

---

# **12. Deliverables**

Students must submit:

* screenshot of interface configuration
* screenshot of routing table
* screenshot of NAT configuration
* screenshot of ACL configuration
* proof of Internet connectivity
* proof of HTTP access
* proof of blocked ports

---

# **13. Reflection Questions**

1. Why is NAT required in this design?
2. Why do we forward to the reverse proxy instead of the web server?
3. Why is the DMZ route required?
4. What is the difference between PAT and static NAT?
5. How does the ACL improve security?

---

# **14. Assessment (Suggested)**

| Criteria                      | Marks |
| ----------------------------- | ----- |
| Interface Config              | 5     |
| Routing (including DMZ route) | 5     |
| NAT Configuration             | 10    |
| ACL Implementation            | 5     |
| Testing                       | 5     |

**Total: 30**

---

# **15. Instructor Notes**

This phase establishes:

* perimeter security
* NAT fundamentals
* traffic flow understanding

---

## ⚠️ Common Student Mistakes

* forgetting the **DMZ route**
* confusing NAT vs ACL
* misconfiguring interface roles
* applying ACL incorrectly

---

