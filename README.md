### This rule set is designed for a Linux system that:

- Allows SSH (port 22) as the primary remote access method.
- Supports basic system functionality (loopback, established/related connections, ICMP).
- Optionally allows common services (web, mail, file sharing, database) while blocking everything else by default.
- Assumes a typical server use case but can be trimmed or expanded based on your needs.

You can remove or comment out rules for services you don’t need.

---

### Full `iptables` Rule Set
```bash
#!/bin/bash

# Flush existing rules to start fresh (use with caution on a live system)
iptables -F
iptables -X

# Set default policies: DROP incoming, ALLOW outgoing
iptables -P INPUT DROP
iptables -P FORWARD DROP  # If routing/NAT isn't needed, drop forwarded traffic
iptables -P OUTPUT ACCEPT # Allow all outbound traffic (can be restricted if needed)

# Allow loopback traffic (essential for local services)
iptables -A INPUT -i lo -j ACCEPT

# Allow established and related connections (keeps ongoing sessions alive)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow ICMP (e.g., ping responses, network diagnostics)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Allow SSH (port 22/TCP) for remote access
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# --- Optional Service Rules (uncomment or remove as needed) ---

# Web Server (HTTP and HTTPS)
#iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
#iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT

# Mail Server (SMTP, Submission, IMAP, IMAPS, POP3, POP3S)
#iptables -A INPUT -p tcp --dport 25 -m conntrack --ctstate NEW -j ACCEPT   # SMTP
#iptables -A INPUT -p tcp --dport 587 -m conntrack --ctstate NEW -j ACCEPT  # Submission
#iptables -A INPUT -p tcp --dport 143 -m conntrack --ctstate NEW -j ACCEPT  # IMAP
#iptables -A INPUT -p tcp --dport 993 -m conntrack --ctstate NEW -j ACCEPT  # IMAPS
#iptables -A INPUT -p tcp --dport 110 -m conntrack --ctstate NEW -j ACCEPT  # POP3
#iptables -A INPUT -p tcp --dport 995 -m conntrack --ctstate NEW -j ACCEPT  # POP3S

# File Sharing - Samba (SMB/CIFS)
#iptables -A INPUT -p udp --dport 137 -m conntrack --ctstate NEW -j ACCEPT  # NetBIOS Name Service
#iptables -A INPUT -p udp --dport 138 -m conntrack --ctstate NEW -j ACCEPT  # NetBIOS Datagram
#iptables -A INPUT -p tcp --dport 139 -m conntrack --ctstate NEW -j ACCEPT  # NetBIOS Session
#iptables -A INPUT -p tcp --dport 445 -m conntrack --ctstate NEW -j ACCEPT  # SMB over TCP

# File Sharing - NFS (basic ports; dynamic ports may need rpcbind config)
#iptables -A INPUT -p tcp --dport 2049 -m conntrack --ctstate NEW -j ACCEPT # NFS
#iptables -A INPUT -p udp --dport 2049 -m conntrack --ctstate NEW -j ACCEPT # NFS

# Database Server (MySQL and PostgreSQL)
#iptables -A INPUT -p tcp --dport 3306 -m conntrack --ctstate NEW -j ACCEPT # MySQL/MariaDB
#iptables -A INPUT -p tcp --dport 5432 -m conntrack --ctstate NEW -j ACCEPT # PostgreSQL

# DHCP Server (if the system assigns IPs; rare for a typical server)
#iptables -A INPUT -p udp --dport 67 -m conntrack --ctstate NEW -j ACCEPT   # DHCP Server

# --- End of Rules ---
# Default INPUT policy is DROP, so anything not explicitly allowed is blocked
```

---

### What This Rule Set Does
1. **Core Functionality**:
   - **Loopback**: Allows all traffic on `lo` for local processes.
   - **Established/Related**: Permits responses to outbound connections (e.g., updates, DNS, NTP) and ongoing sessions.
   - **ICMP**: Allows ping responses and basic network diagnostics.
   - **SSH**: Opens port 22/TCP for remote access.

2. **Optional Services** (commented out by default):
   - Web server (80, 443).
   - Mail server (25, 587, 143, 993, 110, 995).
   - File sharing (Samba: 137-139, 445; NFS: 2049).
   - Databases (3306, 5432).
   - DHCP server (67).

3. **Security**:
   - Default `INPUT` policy is `DROP`, blocking all unlisted inbound traffic.
   - `OUTPUT` is left open (common for servers; can be restricted if needed).
   - `FORWARD` is dropped (assuming no routing/NAT).

---

### How to Use This
1. **Save to a Script**:
   - Copy the code into a file (e.g., `firewall.sh`).
   - Make it executable: `chmod +x firewall.sh`.
   - Run it: `./firewall.sh`.

2. **Customize**:
   - Uncomment the optional service rules you need (e.g., web server ports).
   - Remove rules for services you don’t use.

3. **Persist Rules**:
   - On Debian/Ubuntu: `iptables-save > /etc/iptables/rules.v4`.
   - On CentOS/RHEL: `service iptables save` or integrate with `firewalld`.

4. **Test**:
   - Check rules: `iptables -L -v -n`.
   - Test SSH: `ssh user@host`.
   - Test blocked ports: Try connecting to unopened ports (e.g., `telnet host 80` should fail).

---

### Notes
- **Outbound Traffic**: The `OUTPUT` chain is `ACCEPT` by default, allowing the system to initiate connections (e.g., for updates or DNS). If you want to lock this down, add specific `OUTPUT` rules and set `-P OUTPUT DROP`.
- **Dynamic Protocols**: NFS and FTP may require additional `conntrack` modules (e.g., `nf_conntrack_ftp`) and port ranges for full functionality.
- **Minimal Setup**: If you only need SSH and basic system operation, use just the uncommented rules.

This rule set balances functionality and security for a typical Linux server.
