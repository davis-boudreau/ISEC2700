# **ISEC2700 – Mini Project 3 (MP03-03)**

## **Phase 03: Secure Reverse Proxy (Ubuntu + NGINX + UFW)**

**Course:** ISEC2700 – Intro to Information Security Practices<br>
**Instructor:** Davis Boudreau<br>
**Type:** Mini-Project Phase<br>
**Mode:** Individual<br>
**Estimated Time:** 1–3 hours<br>
**Prerequisite:** MP03-02 completed<br>

---

# **1. Overview / Purpose**

In this phase, you will configure a **secure reverse proxy server** using:

* **Ubuntu Server 24.04**
* **NGINX**
* **UFW (Uncomplicated Firewall)**

---

## 🎯 Your Goals

You will:

* understand what a **reverse proxy** is and why it is used
* install and configure **NGINX**
* configure **UFW host-based firewall**
* allow only required ports (**80 / 443**)
* implement **reverse proxy routing to the web application**
* understand **HTTP/HTTPS termination**

---

# **2. What You Are Building (Architecture)**

---

## 🔐 Role of the Reverse Proxy

The reverse proxy acts as:

* the **only publicly exposed server**
* a **security buffer** between users and backend systems
* a **traffic controller** for web requests

---

## 🧠 Traffic Flow

```text
Internet
   ↓
Router (NAT 80/443)
   ↓
pfSense Firewall (filters traffic)
   ↓
Reverse Proxy (NGINX)
   ↓
Web Application (192.168.40.2)
```

---

## 🔥 Key Security Principle

> 🚨 Backend systems are NEVER directly exposed to the Internet

---

# **3. Key Concepts (Teaching Section)**

---

## 🔹 What is a Reverse Proxy?

A reverse proxy:

* receives client requests
* forwards them to backend servers
* returns the response to the client

---

## 🔹 Benefits

| Benefit         | Description             |
| --------------- | ----------------------- |
| Security        | hides internal systems  |
| Control         | centralizes access      |
| Filtering       | can block/limit traffic |
| TLS termination | handles encryption      |

---

## 🔹 HTTP vs HTTPS Termination

---

### HTTP (Port 80)

* unencrypted traffic

---

### HTTPS (Port 443)

* encrypted traffic (TLS)

---

### 🔐 Termination Concept

The proxy:

* receives HTTPS traffic
* decrypts it
* forwards plain HTTP internally

---

👉 This reduces complexity on backend systems

---

## 🔹 UFW (Host Firewall)

UFW protects the server itself.

| Layer       | Tool    |
| ----------- | ------- |
| Network     | pfSense |
| Host        | UFW     |
| Application | NGINX   |

---

## 🔥 Defense-in-Depth

> Multiple layers of security working together

---

# **4. Network Reference**

| Component  | Address      |
| ---------- | ------------ |
| Proxy eth0 | 192.168.20.2 |
| Proxy eth1 | 192.168.30.1 |
| Web Server | 192.168.40.2 |

---

# **5. Initial Server Setup (Ubuntu)**

---

## 🔧 Step 1 – Access Ubuntu Server

Open console in GNS3

---

## 🔧 Step 2 – Update System

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 🔧 Step 3 – Verify Network

```bash
ip addr
ip route
```

Ensure:

* eth0 → DMZ
* eth1 → internal network

---

# **6. Install NGINX**

---

## 🔧 Step 4 – Install

```bash
sudo apt install nginx -y
```

---

## 🔧 Step 5 – Start Service

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 🔧 Step 6 – Verify

From Chromium:

```text
http://172.16.184.2XX
```

You should see:
👉 NGINX default page

---

# **7. Configure UFW (Host Firewall)**

---

## 🔧 Step 7 – Enable UFW

```bash
sudo ufw enable
```

---

## 🔧 Step 8 – Allow Required Ports

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## 🔧 Step 9 – Deny Everything Else

(Default already deny)

Check:

```bash
sudo ufw status verbose
```

---

## 🧠 Teaching Point

Even if pfSense allows traffic:

👉 UFW can still block it

---

# **8. Configure Reverse Proxy**

---

## 🔧 Step 10 – Edit NGINX Config

```bash
sudo nano /etc/nginx/sites-available/default
```

---

## 🔧 Step 11 – Replace with Reverse Proxy Config

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://192.168.40.2;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🔧 Step 12 – Restart NGINX

```bash
sudo systemctl restart nginx
```

---

## 🧠 What This Does

* receives HTTP requests
* forwards them to:

  ```
  192.168.40.2 (web server)
  ```

---

# **9. Configure HTTPS (Basic Introduction)**

---

## 🔹 For this lab (simplified)

We simulate HTTPS readiness.

```bash
sudo ufw allow 443/tcp
```

Explain:

* real environments use certificates (Let’s Encrypt)
* we are focusing on architecture, not PKI

---

# **10. Validation Testing**

---

## ✅ Test 1 – Proxy Access

From Chromium:

```text
http://172.16.184.2XX
```

Expected:

* content from backend web server

---

## ✅ Test 2 – Backend Isolation

Try accessing directly:

```text
http://192.168.40.2
```

Expected:

* ❌ should NOT be reachable externally

---

## ✅ Test 3 – UFW Status

```bash
sudo ufw status
```

---

# **11. Troubleshooting**

---

## ❌ Cannot reach proxy

* firewall rule missing
* nginx not running

---

## ❌ Proxy not forwarding

* wrong IP in config
* nginx not restarted

---

## ❌ Backend unreachable

* routing issue
* VLAN config issue

---

# **12. Deliverables**

* Screenshot of NGINX config
* Screenshot of UFW status
* Screenshot of working proxy
* Screenshot showing backend isolation
* Short explanation:

Explain:

* reverse proxy purpose
* UFW role
* why backend is hidden

---

# **13. Reflection Questions**

1. Why do we use a reverse proxy instead of exposing the web server directly?
2. What is HTTP/HTTPS termination?
3. Why do we allow only ports 80 and 443?
4. What role does UFW play if pfSense already exists?
5. What would happen if the proxy was compromised?

---

# **14. Assessment (Suggested)**

| Criteria             | Marks |
| -------------------- | ----- |
| NGINX Setup          | 5     |
| Reverse Proxy Config | 10    |
| UFW Configuration    | 5     |
| Testing              | 5     |
| Documentation        | 5     |

**Total: 30**

---

# **15. Instructor Notes**

This phase is where students finally understand:

* layered security
* application exposure control
* real-world architecture patterns

---

## Common Student Mistakes

* confusing proxy IP vs backend IP
* forgetting to restart nginx
* assuming UFW replaces pfSense

---
