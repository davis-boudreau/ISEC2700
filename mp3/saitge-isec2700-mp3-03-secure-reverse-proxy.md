# **ISEC2700 – Mini Project 3 (MP03-03)**

## **Phase 03: Secure Reverse Proxy (Ubuntu Server + Netplan + NGINX + UFW)**

**Course:** ISEC2700 – Intro to Information Security Practices
**Instructor:** Davis Boudreau
**Type:** Mini-Project Phase
**Mode:** Individual
**Estimated Time:** 1–2 hours
**Prerequisite:** MP03-02 completed

---

# **1. Overview / Purpose**

In this phase, you will configure a **secure reverse proxy server** using:

* **Ubuntu Server**
* **netplan** for network configuration
* **NGINX**
* **UFW (Uncomplicated Firewall)**

This server sits between the firewall and the internal application network and acts as the **single public-facing application entry point**.

---

## 🎯 **Your Goals**

You will:

* configure the Ubuntu server’s **two network interfaces**
* implement the correct **netplan configuration**
* add the required **static route to the web application network**
* install and configure **NGINX**
* configure **UFW** to allow only required inbound traffic
* proxy traffic to the internal Django application
* understand **HTTP/HTTPS termination** and **application exposure control**

---

# **2. What You Are Building (Architecture)**

The reverse proxy sits between:

* **pfSense LAN / DMZ side** → `192.168.20.0/24`
* **Core switch VLAN 30 network** → `192.168.30.0/24`

The reverse proxy then forwards traffic to the web application on:

* **VLAN 40 / Web tier** → `192.168.40.0/24`

---

## **Traffic Path**

```text
Internet
   ↓
ISP Router (NAT 80/443)
   ↓
pfSense Firewall
   ↓
Reverse Proxy (192.168.20.2 / 192.168.30.1)
   ↓
CORE-SW (192.168.30.2)
   ↓
Web App (192.168.40.2)
```

---

## **Key Security Principle**

> The reverse proxy is the public application entry point.
> The internal web server should not be the public-facing service.

---

# **3. Why the Network Configuration Matters**

This phase is not only about NGINX.

It is also about making sure the Ubuntu server can correctly route traffic between:

* its **DMZ-side interface**
* its **internal application-side interface**
* the **web application network beyond the core switch**

Without a correct route, the proxy may be able to:

* receive requests from users
  but fail to:
* reach the application at `192.168.40.2`

That creates a frustrating situation where:

* the firewall looks correct
* NGINX looks correct
* the web app is running
* but the reverse proxy still cannot reach the application

---

# **4. Network Reference**

| Interface | Role                         | Address           |
| --------- | ---------------------------- | ----------------- |
| `enp0s4`  | DMZ-side interface           | `192.168.20.2/24` |
| `enp0s5`  | Internal app-entry interface | `192.168.30.1/24` |

---

## **Required Routes**

| Destination       | Next Hop       | Why                                           |
| ----------------- | -------------- | --------------------------------------------- |
| Default route     | `192.168.20.1` | reach upstream networks through pfSense       |
| `192.168.40.0/24` | `192.168.30.2` | reach web application network through CORE-SW |

---

# **5. Key Concepts (Teaching Section)**

## **What is a Reverse Proxy?**

A reverse proxy:

* accepts client web requests
* forwards them to a backend application
* sends the backend response back to the client

Clients do not need to know where the application actually lives.

---

## **Why Use a Reverse Proxy?**

| Benefit         | Description                                                   |
| --------------- | ------------------------------------------------------------- |
| Security        | hides backend application servers                             |
| Control         | centralizes public access                                     |
| Simplicity      | clients only access one front-end service                     |
| Flexibility     | backend apps can move without changing the public entry point |
| TLS termination | HTTPS can be handled at the proxy                             |

---

## **What is HTTP/HTTPS Termination?**

The reverse proxy can:

* accept HTTP/HTTPS from clients
* decrypt HTTPS if configured with certificates
* forward plain HTTP internally to the app

This keeps TLS handling centralized instead of requiring every internal service to manage certificates.

---

## **Why UFW if pfSense Already Exists?**

This is **defense-in-depth**.

| Security Layer    | Tool    |
| ----------------- | ------- |
| Edge firewall     | pfSense |
| Host firewall     | UFW     |
| Application layer | NGINX   |

Even if traffic is allowed through pfSense, UFW can still block it at the Ubuntu host.

---

# **6. Ubuntu Server Network Configuration (Netplan)**

This is the most important addition to this phase.

The Ubuntu reverse proxy uses **netplan** with **networkd**.

---

## **Required Netplan Configuration**

Students must configure the reverse proxy with the following netplan:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    enp0s4:
      dhcp4: no
      addresses:
        - 192.168.20.2/24
      routes:
        - to: default
          via: 192.168.20.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 4.4.4.4

    enp0s5:
      dhcp4: no
      addresses:
        - 192.168.30.1/24
      routes:
        - to: 192.168.40.0/24
          via: 192.168.30.2
```

---

## **What This Configuration Does**

### `enp0s4`

* connects to the **DMZ network**
* gives the server `192.168.20.2/24`
* sets the **default gateway** to `192.168.20.1` (pfSense)

### `enp0s5`

* connects to the **application entry network**
* gives the server `192.168.30.1/24`
* adds a static route so the proxy knows:

> To reach `192.168.40.0/24`, send packets to `192.168.30.2`

This is the route to the core switch, which then routes to the web VLAN.

---

# **7. Step-by-Step Netplan Setup**

## **Step 1 – Open the Netplan File**

On Ubuntu, open the netplan configuration file. The filename may vary depending on the image, but a common path is:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

or

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

---

## **Step 2 – Replace the File Contents**

Paste the required configuration:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    enp0s4:
      dhcp4: no
      addresses:
        - 192.168.20.2/24
      routes:
        - to: default
          via: 192.168.20.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 4.4.4.4

    enp0s5:
      dhcp4: no
      addresses:
        - 192.168.30.1/24
      routes:
        - to: 192.168.40.0/24
          via: 192.168.30.2
```

---

## **Step 3 – Apply the Configuration**

```bash
sudo netplan apply
```

---

## **Step 4 – Verify Interfaces**

```bash
ip addr
```

Expected:

* `enp0s4` = `192.168.20.2/24`
* `enp0s5` = `192.168.30.1/24`

---

## **Step 5 – Verify Routes**

```bash
ip route
```

Expected to include:

* default via `192.168.20.1`
* `192.168.40.0/24 via 192.168.30.2`

---

## **Step 6 – Validate Reachability**

Test the core switch interface:

```bash
ping 192.168.30.2
```

Test the web application network path:

```bash
ping 192.168.40.2
```

Expected:

* reachable, if the web application is present and switch routing/ACLs are configured correctly

---

# **8. Initial System Preparation**

## **Step 7 – Update the Server**

```bash
sudo apt update
sudo apt upgrade -y
```

---

## **Step 8 – Confirm Hostname (Optional but Recommended)**

```bash
hostnamectl set-hostname dmz-proxy
```

---

# **9. Install and Start NGINX**

## **Step 9 – Install NGINX**

```bash
sudo apt install nginx -y
```

---

## **Step 10 – Enable and Start NGINX**

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## **Step 11 – Verify Service Status**

```bash
sudo systemctl status nginx
```

Expected:

* active (running)

---

# **10. Configure UFW (Host-Based Firewall)**

## **Step 12 – Allow Required Inbound Ports**

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## **Step 13 – Enable UFW**

```bash
sudo ufw enable
```

---

## **Step 14 – Verify UFW Rules**

```bash
sudo ufw status verbose
```

Expected:

* `80/tcp` allowed
* `443/tcp` allowed

---

## **Teaching Point**

Even if the pfSense firewall allows traffic through the network, the Ubuntu server still decides what it will accept locally.

That is why UFW is important.

---

# **11. Configure NGINX as a Reverse Proxy**

## **Step 15 – Edit the Default Site**

```bash
sudo nano /etc/nginx/sites-available/default
```

Replace the default content with:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://192.168.40.2;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## **What This Does**

* accepts HTTP requests on the reverse proxy
* forwards them to the web application at `192.168.40.2`
* preserves client-related headers for the backend application

---

## **Step 16 – Test the NGINX Configuration**

```bash
sudo nginx -t
```

Expected:

* syntax is ok
* test is successful

---

## **Step 17 – Restart NGINX**

```bash
sudo systemctl restart nginx
```

---

# **12. HTTPS Discussion and Preparation**

## **Why Allow 443 If the Lab Is Simple?**

Even if the backend application is currently a demo app, the architecture should reflect production-style exposure:

* users should only need HTTP/HTTPS
* all other inbound ports remain closed

For this phase, students should:

* allow TCP 443 in UFW
* understand that HTTPS termination is normally handled here
* know that real certificates would be added in a more advanced lab

---

## **Optional Teaching Note**

If the instructor wants, a later enhancement can include:

* self-signed certificates
* local TLS testing
* full HTTPS termination demonstration

---

# **13. Validation Testing**

## **Test 1 – Confirm the Proxy Can Reach the Web Application**

From the Ubuntu proxy itself:

```bash
curl http://192.168.40.2
```

Expected:

* the web app responds

If this fails, the most likely causes are:

* missing netplan route to `192.168.40.0/24`
* switch ACL issue
* web app not running

---

## **Test 2 – Confirm Public Proxy Access**

From the external test path, browse to:

```text
http://172.16.184.2XX
```

Expected:

* the request reaches the reverse proxy
* the reverse proxy forwards to the web application
* the application page loads

---

## **Test 3 – Confirm UFW is Active**

```bash
sudo ufw status
```

---

## **Test 4 – Confirm Listening Ports**

```bash
ss -tulpn
```

Expected:

* NGINX listening on port 80
* optionally 443 later, if configured

---

# **14. Troubleshooting Guide**

## **Problem: NGINX is running, but the web app does not load**

Check:

* can the proxy ping `192.168.40.2`?
* does `ip route` show `192.168.40.0/24 via 192.168.30.2`?
* is the web application running?
* does the switch allow VLAN 30 → VLAN 40 on TCP 80/443?

---

## **Problem: The proxy cannot reach the web network**

Check the netplan configuration carefully.

Missing this route is a common failure:

```yaml
- to: 192.168.40.0/24
  via: 192.168.30.2
```

Without it, the reverse proxy does not know how to reach the web tier.

---

## **Problem: NGINX configuration fails**

Run:

```bash
sudo nginx -t
```

Fix syntax errors before restarting the service.

---

## **Problem: External users cannot reach the site**

Check:

* router static NAT to `192.168.20.2`
* pfSense WAN rules allowing 80/443 to `192.168.20.2`
* UFW rules
* NGINX status

---

# **15. Deliverables**

Students must submit:

1. screenshot of the Ubuntu netplan file
2. screenshot of `ip addr`
3. screenshot of `ip route` showing:

   * default route
   * route to `192.168.40.0/24 via 192.168.30.2`
4. screenshot of `sudo ufw status`
5. screenshot of the NGINX reverse proxy configuration
6. screenshot of successful `curl http://192.168.40.2` from the proxy
7. screenshot of successful external browser access through `http://172.16.184.2XX`

---

# **16. Reflection Questions**

1. Why does the reverse proxy need two network interfaces?
2. Why is the static route to `192.168.40.0/24` required?
3. What role does the core switch play in reaching the web tier?
4. Why do we use UFW if pfSense already exists?
5. Why is the reverse proxy the correct public entry point instead of exposing the web application directly?

---

# **17. Assessment (Suggested)**

| Criteria                    | Marks |
| --------------------------- | ----: |
| Netplan configuration       |    10 |
| NGINX setup                 |     5 |
| Reverse proxy configuration |     5 |
| UFW hardening               |     5 |
| Testing and validation      |     5 |

**Total: 30 Marks**

---

# **18. Instructor Notes**

This phase is where students begin to understand that application delivery depends on both:

* **security controls**
* **correct routing**

The static route in netplan is a major teaching point because it shows that a server may have multiple interfaces, but still needs routing knowledge to reach non-directly connected networks.

Common student mistakes:

* forgetting to apply netplan
* using the wrong interface names
* forgetting the route to `192.168.40.0/24`
* assuming that being connected to `192.168.30.0/24` automatically means all internal routes are known

---

# **19. Closing Summary for Students**

By the end of MP03-03, your reverse proxy should:

* have two working interfaces
* know how to reach both upstream and internal application networks
* accept inbound HTTP/HTTPS only
* securely forward requests to the internal web application
* act as the only intended public entry point for the application stack

---
