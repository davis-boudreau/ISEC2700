# Using SSH Copy (scp)

---

> **Conventions used**
>
> *   **Linux home**: `~/`
> *   **Windows home (PowerShell)**: `$env:USERPROFILE`
> *   For remote **hostnames** use `remotehost` / `remotewinhost`
> *   Replace `username` with your account on the remote system.

***

## Windows prerequisites (Client & Server)

### To **use `scp` from Windows (PowerShell)**

*   Install **OpenSSH Client** (built-in on Windows 10/11; if missing):
    *   **PowerShell (admin)**:
        ```powershell
        Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
        ```
    *   Verify:
        ```powershell
        scp -V
        ssh -V
        ```

### To **accept `scp` connections to Windows**

*   Install **OpenSSH Server**:
    *   **PowerShell (admin)**:
        ```powershell
        Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
        ```
*   Start and enable the SSH service:
    ```powershell
    Start-Service sshd
    Set-Service -Name sshd -StartupType Automatic
    ```
*   (Optional, if not already present) Enable the firewall rule:
    ```powershell
    if (-not (Get-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -ErrorAction SilentlyContinue)) {
        New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
    } else {
        Enable-NetFirewallRule -Name 'OpenSSH-Server-In-TCP'
    }
    ```
*   Ensure the destination account exists and can sign in via SSH (local or domain), and (optionally) set up `~\.ssh\authorized_keys` for key-based auth.

> **Note on paths** when addressing a **remote Windows host**:
>
> *   Prefer `$env:USERPROFILE\path` (PowerShell expands on the **local** side).
> *   Some remote SSH servers do **not** expand PowerShell variables on the **remote** side. If that happens, use:
>         /c/Users/USERNAME/path
>     or escape backslashes: `C:\\Users\\USERNAME\\path`.

***

## A) Linux ⟷ Linux

### A1. **Local Linux → Remote Linux**

**File**

```bash
scp ~/file.txt username@remotehost:~/file.txt
```

**Directory**

```bash
scp -r ~/myfolder username@remotehost:~/myfolder
```

### A2. **Remote Linux → Local Linux**

**File**

```bash
scp username@remotehost:~/file.txt ~/file.txt
```

**Directory**

```bash
scp -r username@remotehost:~/myfolder ~/myfolder
```

***

## B) Linux ⟷ Windows

### B1. **Local Linux → Remote Windows**

> Remote Windows must have **OpenSSH Server** running (see prerequisites above).

**File (try this first)**

```bash
scp ~/file.txt username@remotewinhost:$env:USERPROFILE/file.txt
```

**If the remote doesn’t expand `$env:USERPROFILE`, use absolute path style:**

```bash
scp ~/file.txt username@remotewinhost:/c/Users/USERNAME/file.txt
```

**Directory**

```bash
scp -r ~/myfolder username@remotewinhost:/c/Users/USERNAME/myfolder
```

### B2. **Remote Windows → Local Windows**

> Execute **on the local Windows machine (PowerShell)**.

**File**

```powershell
scp username@remotewinhost:/c/Users/USERNAME/file.txt $env:USERPROFILE\file.txt
```

**Directory**

```powershell
scp -r username@remotewinhost:/c/Users/USERNAME/myfolder $env:USERPROFILE\myfolder
```

> If the remote Windows host **does** expand variables, you can also use:
>
> ```powershell
> scp username@remotewinhost:$env:USERPROFILE\file.txt $env:USERPROFILE\file.txt
> ```

***

## C) Windows ⟷ Linux

### C1. **Local Windows (PowerShell) → Remote Linux**

**File**

```powershell
scp $env:USERPROFILE\file.txt username@remotehost:~/file.txt
```

**Directory**

```powershell
scp -r $env:USERPROFILE\myfolder username@remotehost:~/myfolder
```

### C2. **Local Linux → Remote Windows**

> Remote Windows must have **OpenSSH Server** running.

**File (PowerShell variable may not expand remotely; prefer absolute path)**

```bash
scp ~/file.txt username@remotewinhost:/c/Users/USERNAME/file.txt
```

**Directory**

```bash
scp -r ~/myfolder username@remotewinhost:/c/Users/USERNAME/myfolder
```

***

## D) Windows ⟷ Windows

### D1. **Local Windows (PowerShell) → Remote Windows**

> Remote Windows must have **OpenSSH Server** running.

**File (recommended absolute path on remote)**

```powershell
scp $env:USERPROFILE\file.txt username@remotewinhost:/c/Users/USERNAME/file.txt
```

**Directory**

```powershell
scp -r $env:USERPROFILE\myfolder username@remotewinhost:/c/Users/USERNAME/myfolder
```

> If the remote expands variables:
>
> ```powershell
> scp $env:USERPROFILE\file.txt username@remotewinhost:$env:USERPROFILE\file.txt
> ```

### D2. **Remote Windows → Local Windows (PowerShell)**

**File**

```powershell
scp username@remotewinhost:/c/Users/USERNAME/file.txt $env:USERPROFILE\file.txt
```

**Directory**

```powershell
scp -r username@remotewinhost:/c/Users/USERNAME/myfolder $env:USERPROFILE\myfolder
```

***

## Useful options & tips

*   Add `-P 22` to specify a **non-default port**:
    ```bash
    scp -P 2222 ~/file.txt user@host:~/
    ```
*   Preserve timestamps/permissions: `-p`
*   Increase robustness for large copies: `-C` (compression), `-v` (verbose)
*   Test SSH first:
    *   From Linux: `ssh username@host`
    *   From Windows (PowerShell): `ssh username@host`

***
