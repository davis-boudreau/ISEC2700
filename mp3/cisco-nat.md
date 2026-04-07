Here’s a clean, practical Cisco IOS NAT setup for your scenario — **G0/0 as the outside interface** and **G0/1 as the inside interface**. I’ll give you a solid baseline config and explain why each piece matters so you can adapt it confidently.

---

## 🌐 Goal  
You want a Cisco router performing NAT with:

- **GigabitEthernet0/0 → Outside (WAN)**
- **GigabitEthernet0/1 → Inside (LAN)**

This is the classic home/office edge-router setup.

---

## 🔧 Example NAT Configuration (Cisco IOS)

```plaintext
! Assign IP addresses to interfaces
interface GigabitEthernet0/0
 description WAN / Outside
 ip address <YOUR_PUBLIC_IP> <MASK>
 ip nat outside
 no shutdown

interface GigabitEthernet0/1
 description LAN / Inside
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown

! Define which inside addresses are allowed to be NATed
access-list 1 permit 192.168.1.0 0.0.0.255

! NAT Overload (PAT) for internet access
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

---

## 🧠 What Each Part Does

### **1. Interface Roles**
- `ip nat inside` → tells the router this interface is the private LAN  
- `ip nat outside` → tells the router this interface faces the internet

### **2. Access List**
ACL 1 defines which internal IPs are allowed to be translated.

### **3. NAT Overload**
This line is the magic:

```plaintext
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

It means:
- Translate all inside addresses in ACL 1
- Use the IP of G0/0 as the public address
- Use port numbers to multiplex many internal hosts (PAT)

---

## 🚀 Optional Enhancements

### **DHCP for LAN**
```plaintext
ip dhcp pool LAN
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8 1.1.1.1
```

### **Static NAT (Port Forwarding)**
Example: forward port 80 to 192.168.1.10

```plaintext
ip nat inside source static tcp 192.168.1.10 80 interface GigabitEthernet0/0 80
```

---

## If you want, I can help you:
- Build a full working config for your exact IPs  
- Add firewall rules (ACLs)  
- Add port forwarding  
- Troubleshoot NAT issues  

Just tell me what your WAN IP setup looks like (DHCP or static) and what devices you’re connecting on the inside.