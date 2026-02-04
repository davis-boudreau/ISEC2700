# 🧪 Workshop: Network File Sharing with NFS (Linux & Windows Clients)

---

## 1. Assignment Details

| Item           | Description                                                |
| -------------- | ---------------------------------------------------------- |
| Course         | OSYS / NETW / ISEC (Instructor to specify)                 |
| Workshop Title | Network File Sharing with NFS                              |
| Type           | Hands-On Infrastructure Workshop                           |
| Delivery       | In-Class / Guided Lab                                      |
| Duration       | 2.5–3 hours                                                |
| Weight         | Practice / Learning Activity (Not Graded unless specified) |
| Prerequisites  | Linux fundamentals, basic networking, SSH access           |
| Tools          | Ubuntu Server 24.04, Linux client VM, Windows 10/11        |
| Submission     | Screenshots + reflection (if required)                     |

---

## 2. Overview / Purpose / Objectives

Modern enterprise environments rely heavily on **centralized file storage** that can be securely accessed across the network.

**Network File System (NFS)** is a widely used Linux-based file-sharing protocol that allows multiple systems to mount and access shared directories as if they were local storage.

In this workshop, students will:

* Configure an **NFS server on Ubuntu**
* Export shared directories securely
* Mount NFS shares on:

  * A **Linux client**
  * A **Windows client**
* Understand how permissions, UID/GID mapping, and network trust affect access
* Connect NFS concepts to **security architecture principles**

---

## 3. Learning Outcomes Addressed

By the end of this workshop, students will be able to:

* Configure and manage a Linux file-sharing service
* Apply correct Linux permissions to shared resources
* Mount and persist network storage on client systems
* Evaluate security risks associated with network file sharing
* Explain the role of NFS within enterprise infrastructure

---

## 4. Assignment Description / Use Case

### Scenario

You are working as a junior systems administrator.

Your organization requires:

* A **central Linux file server**
* Multiple Linux systems accessing shared project files
* Optional access from Windows workstations

You are tasked with:

* Deploying an **NFS server**
* Exporting a secure shared directory
* Connecting Linux and Windows clients
* Ensuring data access works as expected

---

## 5. Tasks / Instructions

### Part A — NFS Server Configuration (Ubuntu)

1. Install NFS server packages
2. Create shared directories
3. Configure export rules
4. Apply permissions
5. Start and verify the NFS service

---

### Part B — Linux Client Configuration

1. Install NFS client utilities
2. Mount the NFS share
3. Test read/write access
4. Configure persistent mounts

---

### Part C — Windows Client Configuration

1. Enable Windows NFS feature
2. Configure UID/GID mapping
3. Mount NFS share
4. Verify file access

---

### Part D — Validation & Troubleshooting

1. Test file creation
2. Verify permissions
3. Troubleshoot common errors

---

## 6. Deliverables

Students may be asked to submit:

* Screenshot of:

  * NFS exports
  * Mounted shares (Linux)
  * Windows NFS mapping
* Short reflection:

  * What worked?
  * What failed initially?
  * Why permissions matter with NFS?

---

## 7. Reflection Questions

1. Why does NFS rely heavily on UID/GID matching?
2. What security risks exist with NFS?
3. Why should NFS not be exposed directly to the public internet?
4. How does NFS align with **least privilege**?
5. When would SMB be preferred over NFS?

---

## 8. Assessment & Rubric (Optional)

| Criteria              | Excellent            | Satisfactory      | Needs Improvement |
| --------------------- | -------------------- | ----------------- | ----------------- |
| NFS Server Setup      | Fully functional     | Minor issues      | Not functional    |
| Linux Client Mount    | Persistent & working | Temporary only    | Failed            |
| Windows Client Access | Successful           | Partial           | Failed            |
| Permissions           | Correct & secure     | Overly permissive | Incorrect         |
| Understanding         | Clear explanation    | Basic             | Limited           |

---

## 9. Submission Guidelines

* Combine screenshots into a single PDF or document
* Name file using:

  ```
  studentid_nfs_workshop.pdf
  ```
* Submit via LMS if required

---

## 10. Resources / Equipment

* Ubuntu Server 24.04 ISO
* Linux desktop VM (Ubuntu recommended)
* Windows 10/11 Pro or Education
* SSH client (PuTTY or OpenSSH)
* Internet access for package installation

---

## 11. Academic Policies

Refer to institutional academic integrity and lab conduct policies.

---

## 12. Copyright Notice

© Nova Scotia Community College
Educational use only.

---

# 🔧 Elaborate Tutorial

---

# PART 1 — NFS SERVER SETUP (Ubuntu 24.04)

---

## Step 1 — Install NFS Server

On the Ubuntu **server**:

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

Verify service:

```bash
systemctl status nfs-kernel-server
```

---

## Step 2 — Create Shared Directory

Example:

```bash
sudo mkdir -p /srv/nfs/shared
```

Set ownership:

```bash
sudo chown nobody:nogroup /srv/nfs/shared
sudo chmod 777 /srv/nfs/shared
```

> ⚠️ 777 is acceptable **for learning only** — not production.

---

## Step 3 — Configure Exports

Edit:

```bash
sudo nano /etc/exports
```

Add:

```
/srv/nfs/shared 192.168.10.0/24(rw,sync,no_subtree_check)
```

Explanation:

| Option           | Meaning              |
| ---------------- | -------------------- |
| rw               | Read/write           |
| sync             | Writes immediately   |
| no_subtree_check | Improves reliability |

Apply exports:

```bash
sudo exportfs -ra
```

Verify:

```bash
sudo exportfs -v
```

---

## Step 4 — Firewall Configuration

If UFW is enabled:

```bash
sudo ufw allow from 192.168.10.0/24 to any port nfs
```

Or allow NFS service:

```bash
sudo ufw allow nfs
```

---

## Step 5 — Confirm NFS Ports

```bash
sudo ss -tulnp | grep nfs
```

NFS uses:

* TCP/UDP 2049
* RPC services

---

# PART 2 — LINUX CLIENT SETUP

---

## Step 1 — Install NFS Client

```bash
sudo apt update
sudo apt install nfs-common -y
```

---

## Step 2 — Create Mount Point

```bash
sudo mkdir -p /mnt/nfs
```

---

## Step 3 — Mount NFS Share

```bash
sudo mount 192.168.10.10:/srv/nfs/shared /mnt/nfs
```

Verify:

```bash
df -h
```

Test:

```bash
touch /mnt/nfs/testfile.txt
ls -l /mnt/nfs
```

---

## Step 4 — Persistent Mount (fstab)

Edit:

```bash
sudo nano /etc/fstab
```

Add:

```
192.168.10.10:/srv/nfs/shared  /mnt/nfs  nfs  defaults  0  0
```

Test safely:

```bash
sudo mount -a
```

---

# PART 3 — WINDOWS CLIENT SETUP

> ⚠️ Windows NFS is available only on **Pro, Education, or Enterprise** editions.

---

## Step 1 — Enable NFS Client Feature

### GUI Method

1. Control Panel
2. Programs and Features
3. Turn Windows features on or off
4. Enable:

   * **Services for NFS**
   * Client for NFS

Reboot.

---

## Step 2 — Configure UID/GID Mapping

Windows does not use Linux UIDs by default.

Open **PowerShell as Administrator**:

```powershell
nfsadmin client stop
nfsadmin client localhost config fileaccess=755 SecFlavors=AUTH_SYS
nfsadmin client start
```

Set default UID/GID:

```powershell
reg add HKLM\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default /v AnonymousUid /t REG_DWORD /d 65534 /f
reg add HKLM\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default /v AnonymousGid /t REG_DWORD /d 65534 /f
```

Restart NFS client service.

---

## Step 3 — Mount NFS Share

```powershell
mount \\192.168.10.10\srv\nfs\shared Z:
```

Check:

```powershell
dir Z:
```

You should now see the same files created from Linux.

---

# PART 4 — Validation

Perform these tests:

✅ Create file from Linux
✅ Modify file from Windows
✅ View same file from server

This confirms **shared storage consistency**.

---

# PART 5 — Security Discussion (Critical)

NFS security risks include:

* No encryption (v3)
* Trust-based authentication
* UID/GID spoofing
* Network sniffing

Mitigations:

* Use **private networks only**
* Combine with **firewalls**
* Use **NFSv4 with Kerberos** (enterprise)
* Apply **least privilege exports**
* Never expose NFS to the internet

---

# 🔐 Security Architecture Mapping

| Principle            | NFS Example                 |
| -------------------- | --------------------------- |
| Least Privilege      | Limit export subnets        |
| Defense in Depth     | Firewall + permissions      |
| Separation of Duties | Storage vs user systems     |
| Security by Design   | Internal-only access        |
| KISS                 | Simple, predictable exports |

---
