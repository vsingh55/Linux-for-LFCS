# Linux Networking Comprehensive Guide

## **1. IPv4, IPv6, and CIDR**
### **Key Concepts**
- **IPv4**: 32-bit address (e.g., `192.168.1.1`), ranges from `0.0.0.0` to `255.255.255.255`.
- **IPv6**: 128-bit address in hexadecimal (e.g., `2001:0db8::ff00:42:8329`).
- **CIDR**: Subnet notation (e.g., `192.168.1.0/24`). The prefix (e.g., `/24`) defines the network portion.

### **Commands & Files**
```bash
# Check IP addresses
ip a       # IPv4/IPv6 addresses
ip -6 addr # IPv6 only

# Subnet calculation tools
nslookup example.com  # DNS resolution
```

---

## **2. Configuring IPv4/IPv6 and Hostname Resolution**
### **Key Files**
- `/etc/network/interfaces` (Legacy)  
- `/etc/netplan/*.yaml` (Ubuntu 18.04+)  
- `/etc/hosts` (Local hostname resolution)  
- `/etc/resolv.conf` (DNS servers)  

### **Commands**
```bash
# Configure static IP (NetPlan example)
sudo nano /etc/netplan/01-netcfg.yaml
# Apply changes
sudo netplan apply

# Set hostname
sudo hostnamectl set-hostname myserver
```

---

## **3. Network Services Management**
### **Check Services**
```bash
# List listening ports
ss -tunlp       # Modern (shows PID/process)
netstat -tunlp  # Legacy

# Check service status
systemctl status sshd

# Start/stop services
sudo systemctl start nginx
sudo systemctl stop nginx
```

### **Process Mapping**
```bash

ss -tunlp | grep :22 # Find process using a port
ps aux | grep sshd  # Find PID of SSH daemon
lsof -p <PID>  # List files opened by a process
```

---

## **4. Bridging and Bonding**
### Bridging
**Purpose:** Connects multiple network interfaces to act as a single Layer 2 (Data Link) network.

**Use Case:** Commonly used in virtualization to allow VMs to access the host's network.

**Example:** A bridge (br0) connects eth0 and eth1, enabling devices on both interfaces to communicate as if on the same network.
### Bonding
**Purpose:** Combines multiple network interfaces for redundancy or increased bandwidth.

**Use Case:** High availability or performance in servers.

**Bonding Modes**

| Mode | Name                      | Description                          |
|------|---------------------------|--------------------------------------|
| 0    | Round-robin               | Distributes traffic sequentially     |
| 1    | Active-backup             | Failover between interfaces          |
| 4    | 802.3ad (LACP)            | Aggregates bandwidth                 |
| 6    | Adaptive load balancing   | Balances outgoing/incoming traffic   |


>Both are configured using tools like **netplan** or **nmcli**.

### **Commands**
```bash
# Create a bond (using netplan)
network:
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: 802.3ad
```

---

## **5. Firewalls and Packet Filtering**
### **UFW (Uncomplicated Firewall)**
```bash
sudo ufw allow 22        # Allow SSH
sudo ufw enable          # Enable firewall
sudo ufw status numbered # List rules
```

### **iptables**
```bash
# Allow port 80
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Save rules
sudo iptables-save > /etc/iptables/rules.v4

# Port forwarding (NAT)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80
sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

---

## **6. Port Redirection & NAT**
### **Enable IP Forwarding**
```bash
# Edit sysctl.conf
sudo nano /etc/sysctl.d/99-sysctl.conf
# Add:
net.ipv4.ip_forward=1

# Apply changes
sudo sysctl -p
```

### **Persistent Rules**
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## **7. Reverse Proxies & Load Balancers (Nginx)**
### **Reverse Proxy Config**
```nginx
# /etc/nginx/sites-available/proxy.conf
server {
    listen 80;
    location / {
        proxy_pass http://backend_server;
        include proxy_params;
    }
}
```

### **Load Balancer Config**
```nginx
upstream backend {
    least_conn;
    server 10.0.0.1:80 weight=3;
    server 10.0.0.2:80;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### **Enable Config**
```bash
sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## **8. Time Synchronization**
### **Configure NTP**
```bash
# Set timezone
sudo timedatectl set-timezone America/New_York

# Enable NTP
sudo timedatectl set-ntp true
systemctl status systemd-timesyncd

# Edit NTP servers
sudo nano /etc/systemd/timesyncd.conf
# Add:
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org
```

---
## **Tips for LFCS Exam**
1. **Troubleshooting**: Use `ip a`, `ip r`, and `ping` to diagnose connectivity.
2. **Firewalls**: Always check UFW/iptables rules if a service is unreachable.
3. **Persistence**: Ensure changes to `iptables` or network configs are saved and applied.



## **Key Processes Explained**  

### **1. Configuring Static IP with NetPlan**  
**Process**:  
1. **Edit NetPlan Config**:  
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```  
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       enp0s3:               # Interface name (use `ip a` to check)
         addresses: 
           - 192.168.1.10/24
         gateway4: 192.168.1.1
         nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
   ```  
2. **Apply Changes**:  
   ```bash
   sudo netplan apply
   ```  
   - **Troubleshoot**: Use `sudo netplan --debug apply` for detailed logs.  

---

### **2. Bonding Interfaces (Mode 4: LACP)**  
**Process**:  
1. **Edit NetPlan Config**:  
   ```yaml
   network:
     version: 2
     bonds:
       bond0:
         interfaces: [enp0s3, enp0s4]
         parameters:
           mode: 802.3ad    # LACP
           lacp-rate: fast  # Aggregation speed
     ethernets:
       enp0s3: {}
       enp0s4: {}
   ```  
2. **Apply**:  
   ```bash
   sudo netplan apply
   ```  
3. **Verify**:  
   ```bash
   cat /proc/net/bonding/bond0
   ```  

---

### **3. Port Forwarding with iptables**  
**Process**:  
1. **Enable IP Forwarding**:  
   ```bash
   sudo nano /etc/sysctl.d/99-sysctl.conf
   # Add: net.ipv4.ip_forward=1
   sudo sysctl -p
   ```  
2. **Redirect Port 8080 → 80**:  
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80
   sudo iptables -t nat -A POSTROUTING -j MASQUERADE
   ```  
3. **Save Rules**:  
   ```bash
   sudo apt install iptables-persistent
   sudo netfilter-persistent save
   ```  

---

### **4. Diagnosing Service Failures**  
**Process**:  
1. **Check Service Status**:  
   ```bash
   systemctl status nginx
   ```  
2. **View Logs**:  
   ```bash
   journalctl -u nginx --since "10 minutes ago"
   ```  
3. **Test Port Conflicts**:  
   ```bash
   ss -tunlp | grep :80  # Check if port 80 is already in use
   ```  
---

## **Final Tips for LFCS**  
1. **Practice CLI Tools**: Master `ip`, `ss`, `journalctl`, and `systemctl`.  
2. **Know Key Files**: `/etc/netplan/*.yaml`, `/etc/resolv.conf`, `/etc/ufw/`.  
3. **Test Configurations**: Always verify changes (e.g., `ping`, `curl`).  

Good luck! 🚀

