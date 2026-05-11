# 🚩 CTF Challenge Writeups

> Selected hands-on CTF solutions from the Cyber Security Professional Course practical assignments. All challenges completed on real lab environments using Kali Linux.

---

## 📋 Index

| # | Challenge | Category | Tools | Status |
|---|-----------|----------|-------|--------|
| 01 | [Hidden Image & Flag Extraction from PCAP](#01---hidden-image--flag-extraction-from-pcap) | Network Forensics | Wireshark | ✅ |
| 02 | [Netcat Flag Capture — Port Listener](#02---netcat-flag-capture--port-listener) | Network Exploitation | Netcat | ✅ |
| 03 | [SSH + Snort IDS — Flag Capture](#03---ssh--snort-ids--flag-capture) | Network Security | SSH, Snort | ✅ |
| 04 | [SUID Bit Executables — Privilege Escalation Recon](#04---suid-bit-executables--privilege-escalation-recon) | Privilege Escalation | Linux CLI | ✅ |
| 05 | [SCP — Secure File Transfer Between Machines](#05---scp--secure-file-transfer-between-machines) | Network | SCP, SSH | ✅ |
| 06 | [SSH Key-Based Authentication Setup](#06---ssh-key-based-authentication-setup) | Network Security | ssh-keygen | ✅ |

---

## 01 - Hidden Image & Flag Extraction from PCAP

**Category:** Network Forensics | **Tool:** Wireshark

### Objective
Find and extract a hidden image file from a network packet capture and retrieve the embedded flag.

### Steps
```
1. Opened capture file in Wireshark
2. File → Export Objects → HTTP
3. Filtered exported objects for image files
4. Previewed extracted image — flag embedded inside
```

### Result
> **Flag:** `HSCS01{bfd3f4c834aac369f9cc4b458f24feb0}`
> **File type:** `.png` — hidden inside HTTP traffic

### Key Learnings
- Wireshark's Export Objects feature reconstructs files transmitted over HTTP — images, documents, executables can all be recovered
- Unencrypted HTTP exposes every file transferred — always use HTTPS
- Network forensics can recover evidence even after the fact from saved captures
- This technique is used in real incident response to identify data exfiltration

### Mitigation
- Enforce HTTPS across all web traffic
- Use TLS inspection at the network boundary
- Monitor for unusual file transfers using DLP tools

---

## 02 - Netcat Flag Capture — Port Listener

**Category:** Network Exploitation | **Tool:** Netcat

### Objective
Start a Netcat listener on a specific port on a target machine and capture the incoming flag.

### Steps
```bash
# Start listener on target machine
nc -lvp 3535

# Wait for incoming connection — flag delivered automatically
```

### Result
> **Flag:** `HSCS01{7671baea4a2d5ef812ad70bac1073cb4}`

### Key Learnings
- Netcat (`nc`) is the Swiss Army knife of networking — used for listening, connecting, file transfer, and port scanning
- `-l` = listen mode, `-v` = verbose output, `-p` = specify port
- Open port listeners are a common **persistence and backdoor technique** used by attackers after initial compromise
- Any unexpected listening port on a system should be treated as suspicious

### Mitigation
- Regularly audit open ports using `netstat -tulpn` or `ss -tulpn`
- Implement host-based firewall rules to block unauthorized listeners
- Use EDR/IDS tools to detect unexpected Netcat usage

---

## 03 - SSH + Snort IDS — Flag Capture

**Category:** Network Security | **Tools:** SSH, Snort

### Objective
SSH into a target machine using key-based authentication, start Snort IDS in console mode, then trigger an alert via ping to capture the flag.

### Steps
```bash
# Step 1: Connect via SSH using private key authentication
ssh user@[target] -i [keyfile]

# Step 2: Start Snort IDS listening on network interface
sudo snort -A console -i eth0 -c /etc/snort/snort.conf

# Step 3: From attacker machine — trigger the Snort rule
ping [target-ip]

# Flag appears in Snort console output
```

### Result
> **Flag:** `HSCS01{2d787f26e978ecc04758420146c7ac34}`

### Key Learnings
- **Snort** is an open-source Network Intrusion Detection System (NIDS) — used widely in enterprise environments
- `-A console` mode outputs alerts in real time to terminal — useful for testing and monitoring
- Even a simple ICMP ping can trigger a Snort rule if configured correctly — demonstrates how IDS rules work
- SSH key authentication (`-i keyfile`) eliminates password brute force risk entirely

### Mitigation
- Deploy Snort or Suricata as NIDS on critical network segments
- Write custom rules for environment-specific threat detection
- Combine IDS alerts with SIEM for centralized monitoring

---

## 04 - SUID Bit Executables — Privilege Escalation Recon

**Category:** Privilege Escalation | **Tool:** Linux CLI

### Objective
Find all executables with the SUID bit set on a Linux system — a key step in privilege escalation reconnaissance.

### Command
```bash
find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null
```

### Understanding the Command
```
find /          → search entire filesystem
-perm -4000     → find files with SUID bit set
-type f         → files only (not directories)
-exec ls -la {} → show detailed permissions for each match
2>/dev/null     → suppress permission errors
```

### How to Identify SUID Files
In the output, look for `s` in the owner execute position:
```
-rwsr-xr-x  → SUID set (runs as file owner, often root)
-rwxr-xr-x  → Normal executable (no SUID)
```

### Key Learnings
- SUID files execute with the **file owner's permissions** — if owned by root, any user can run it as root
- Finding misconfigured SUID binaries is a standard post-exploitation step
- Common legitimate SUID binaries: `passwd`, `sudo`, `ping` — anything else warrants investigation
- Tools like GTFOBins document how SUID binaries can be abused for privilege escalation

### Mitigation
- Audit SUID files regularly and remove unnecessary SUID bits
- Use `chmod u-s filename` to remove SUID from files that don't need it
- Monitor SUID changes using file integrity monitoring tools

---

## 05 - SCP — Secure File Transfer Between Machines

**Category:** Network | **Tools:** SCP, SSH

### Objective
Create a file on a remote machine and securely transfer it to a local machine using SCP.

### Steps
```bash
# Step 1: Create a file on remote machine
cd /tmp
echo "test content" >> transferfile.txt

# Step 2: Ensure SSH service is running on destination machine
sudo service ssh start

# Step 3: Transfer file from remote to local using SCP
scp transferfile.txt user@[destination-ip]:/home/user/
```

### SCP Syntax Reference
```
scp [source-file] [user]@[destination-ip]:[destination-path]

# Remote to local:
scp user@remote:/path/to/file /local/destination/

# Local to remote:
scp /local/file user@remote:/path/to/destination/
```

### Key Learnings
- SCP (Secure Copy Protocol) uses SSH encryption — all file data is encrypted in transit
- Always preferred over FTP which transmits data in plaintext
- Understanding SCP is essential for incident response, log collection, and forensic data transfer
- Can transfer entire directories using `-r` flag

### Mitigation
- Disable FTP entirely — replace with SCP or SFTP
- Restrict SCP access using SSH `AllowUsers` configuration
- Log all SCP transfers for audit purposes

---

## 06 - SSH Key-Based Authentication Setup

**Category:** Network Security | **Tools:** SSH, ssh-keygen

### Objective
Set up passwordless SSH login using RSA key pair authentication — replacing password-based login with cryptographic keys.

### Steps
```bash
# Step 1: Generate RSA key pair on local machine
ssh-keygen -t rsa
# Creates: ~/.ssh/id_rsa (private) and ~/.ssh/id_rsa.pub (public)

# Step 2: Copy public key to target machine
ssh-copy-id user@[target-ip]
# Appends public key to target's ~/.ssh/authorized_keys

# Step 3: Login without password
ssh user@[target-ip]
# Authenticates using private key — no password prompt
```

### How It Works
```
Local Machine                    Remote Machine
┌─────────────────┐              ┌─────────────────┐
│  Private Key    │              │  Public Key     │
│  (~/.ssh/id_rsa)│◄────────────►│  (authorized_   │
│  NEVER shared   │  Handshake  │   keys)          │
└─────────────────┘              └─────────────────┘
```

### Key Learnings
- Private key stays on your machine — **never share it**
- Public key can be safely placed on any server you want to access
- Eliminates password brute force attacks entirely
- `ssh-copy-id` automates adding the public key to the right location

### Hardening After Key Setup
```bash
# Disable password authentication in /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no

# Restart SSH service
sudo systemctl restart sshd
```

### Mitigation Value
- Key-based auth + disabled password login = SSH brute force attacks become impossible
- Combined with fail2ban for complete SSH hardening

---

## 🛠️ Tools Used Across All Challenges

`Wireshark` `Netcat` `SSH` `SCP` `Snort` `Kali Linux` `ssh-keygen` `find`

---

*Part of the 6-month Cyber Security Professional Course — CEH certification in progress (July 2026)*
