### **Scenario: User Creation**
**Task**:  
Create a user "backupadmin" with:  
- UID 3000  
- Home directory `/opt/backup`  
- Default shell `/sbin/nologin`  
- Secondary group membership "sysadmin"

<details>
<summary>Solution</summary>

```bash
# Create sysadmin group if not exists
sudo groupadd sysadmin

# Create user with custom UID and home directory
sudo useradd -u 3000 -d /opt/backup -m -s /sbin/nologin backupadmin

# Add to secondary group
sudo usermod -aG sysadmin backupadmin

# Verify
id backupadmin
ls -ld /opt/backup
```

<details>
<summary>Verification</summary>

- Check `/etc/passwd` for UID 3000  
- Confirm shell in `grep backupadmin /etc/passwd`

</details>
</details>

---

### **Scenario: Password Policy**
**Task**:  
Enforce password policy for user "devuser":  
- Password expires every 45 days  
- Warn user 7 days before expiration  
- Account expires on 2024-12-31

<details>
<summary>Solution</summary>

```bash
# Set password aging
sudo chage -M 45 -W 7 -E 2024-12-31 devuser

# Verify
sudo chage -l devuser
```

<details>
<summary>Key Files</summary>

- Changes reflected in `/etc/shadow`

</details>
</details>

---

### **Scenario: Group Management**
**Task**:  
1. Create group "developers" with GID 4000  
2. Make "devuser" primary group "developers"  
3. Allow "sysadmin" group members to add users to "developers"

<details>
<summary>Solution</summary>

```bash
# Create group with GID
sudo groupadd -g 4000 developers

# Change primary group
sudo usermod -g developers devuser

# Delegate group management
sudo gpasswd -A sysadmin developers

# Verify
groups devuser
getent group developers
```

<details>
<summary>Note</summary>

Members of "sysadmin" can now use `gpasswd -a user developers`

</details>
</details>

---

### **Scenario: Troubleshooting Permissions**
**Problem**:  
User "audituser" can't access `/var/log/audit` even though:  
```bash
drwxr-x--- 2 root audit 4096 Jun 10 09:00 /var/log/audit
```
User is in "audit" group but still gets "Permission denied"

<details>
<summary>Solution</summary>

1. Check effective group membership:
```bash
id audituser
```
2. If user needs to assume group identity:
```bash
newgrp audit
```
3. Alternative solution (requires logout/login):
```bash
sudo usermod -aG audit audituser
```

<details>
<summary>Key Command</summary>

`ls -ld /var/log/audit` to verify group ownership

</details>
</details>

---

### **Scenario: Sudo Access**
**Task**:  
Allow "devuser" to run ONLY `systemctl restart nginx` as root without password

<details>
<summary>Solution</summary>

```bash
sudo visudo
```
Add this line:
```bash
devuser ALL=(root) NOPASSWD: /bin/systemctl restart nginx
```

<details>
<summary>Verification</summary>

```bash
sudo -l -U devuser
```

</details>
</details>

---

### **Scenario: Resource Limits**
**Task**:  
Limit user "testuser" to:  
- Max 100 concurrent processes  
- Max 50MB file size

<details>
<summary>Solution</summary>

Edit `/etc/security/limits.conf`:
```bash
testuser hard nproc 100
testuser hard fsize 51200
```

<details>
<summary>Verification</summary>

```bash
su - testuser
ulimit -a
```

</details>
</details>

---

### **Scenario: System User**
**Task**:  
Create a system user "nginx" without home directory for running web server

<details>
<summary>Solution</summary>

```bash
sudo useradd -r -s /sbin/nologin nginx

# Verify
grep nginx /etc/passwd
```

<details>
<summary>Flags</summary>

`-r` = system user, `-s` = set login shell

</details>
</details>

---

### **Scenario: Restrict SSH Access to a Specific IP**
**Task**: Allow SSH (port 22) only from `192.168.1.100`.  

<details>
<summary>Solution</summary>

```bash
sudo ufw allow from 192.168.1.100 to any port 22
sudo ufw enable
```
</details>

---

### **Scenario: Configure a Reverse Proxy for a Web Server**
**Task**: Redirect traffic from `Nginx (port 80)` to an internal server at `10.0.0.5:8080`.  

<details>
<summary>Solution</summary>

1. **Edit Nginx Config**:  
   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://10.0.0.5:8080;
       }
   }
   ```  
2. **Reload Nginx**:  
   ```bash
   sudo systemctl reload nginx
   ```

</details>

---

### **Scenario: Troubleshoot DNS Resolution Failure**
**Task**: Fix DNS resolution for `example.com`.  

<details>
<summary>Solution</summary>

1. Check `/etc/resolv.conf` for valid nameservers.  
2. Test DNS lookup:  
   ```bash
   nslookup example.com 8.8.8.8  # Use Google DNS
   ```  
3. If resolved, update nameserver in `/etc/resolv.conf`.  

</details>

---

### **Scenario: Set Up IPv6 Address**
**Task**: Assign `2001:db8::1/64` to interface `enp0s3`.  

<details>
<summary>Solution</summary>

1. **Edit NetPlan**:  
   ```yaml
   network:
     version: 2
     ethernets:
       enp0s3:
         addresses:
           - 2001:db8::1/64
   ```  
2. **Apply**:  
   ```bash
   sudo netplan apply
   ```

</details>

---

### **Scenario: Bond Two Interfaces in Active-Backup Mode**
**Task**: Ensure failover if `enp0s3` fails.  

<details>
<summary>Solution</summary>

1. **Edit NetPlan**:  
   ```yaml
   bonds:
     bond0:
       interfaces: [enp0s3, enp0s4]
       parameters:
         mode: active-backup
   ```  
2. **Apply**:  
   ```bash
   sudo netplan apply
   ```

</details>

---

### **Scenario: Block ICMP (Ping) Requests**
**Task**: Prevent the server from responding to pings.  

<details>
<summary>Solution</summary>

```bash
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
# Make permanent:
sudo nano /etc/sysctl.conf  # Add: net.ipv4.icmp_echo_ignore_all=1
```

</details>

---

### **Scenario: Diagnose High Network Latency**
**Task**: Identify delays in reaching `example.com`.  

<details>
<summary>Solution</summary>

1. **Trace Route**:  
   ```bash
   traceroute example.com
   ```  
2. **Test Packet Loss**:  
   ```bash
   ping -c 100 example.com
   ```

</details>

---

### **Scenario: Configure NTP for Time Sync**
**Task**: Sync time with `pool.ntp.org`.  

<details>
<summary>Solution</summary>

1. **Edit Config**:  
   ```bash
   sudo nano /etc/systemd/timesyncd.conf
   # Add: NTP=pool.ntp.org
   ```  
2. **Restart Service**:  
   ```bash
   sudo systemctl restart systemd-timesyncd
   ```

</details>

---

### **Scenario: Allow HTTP/HTTPS Only**
**Task**: Block all traffic except ports 80/443.  

<details>
<summary>Solution</summary>

```bash
sudo ufw default deny incoming
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

</details>

---

### **Scenario: Resolve "Port Already in Use" Error**
**Task**: Free port 80 occupied by an unknown process.  

<details>
<summary>Solution</summary>

1. **Find PID**:  
   ```bash
   sudo ss -tunlp | grep :80
   ```  
2. **Terminate Process**:  
   ```bash
   sudo kill -9 <PID>
   ```

</details>

---

### **Scenario: File Management in `/opt/findme/`**
**Task**:  
1. Find files with executable permission for the owner. Redirect output to `/opt/foundthem.txt`.  
2. Delete files with SETUID permission.  
3. Copy files larger than 1KB to `/opt/`.

<details>
<summary>Solution</summary>

```bash
# Executable files
find /opt/findme/ -type f -perm /u=x > /opt/foundthem.txt

# Delete SETUID files
sudo find /opt/findme/ -type f -perm /u=s -exec rm {} \;

# Copy files >1KB
sudo find /opt/findme/ -type f -size +1k -exec cp {} /opt/ \;
```
</details>

---

### **Scenario: Backup Script and Cron Job**
**Task**:  
1. Create a script `/opt/script.sh` to recursively copy `/var/www/` to `/opt/www-backup/`.  
2. Schedule the script to run daily at 4 AM via system-wide cron.

<details>
<summary>Solution</summary>

```bash
#!/bin/bash
# Script content
cp -rp /var/www/* /opt/www-backup/
```

```bash
# Make script executable
sudo chmod +x /opt/script.sh

# Add to system-wide cron (edit /etc/crontab)
0 4 * * * root /opt/script.sh
```
</details>

---

### **Scenario: User Limits**
**Task**:  
1. Set a **hard limit** of 30 processes for user `john`.  
2. Set a **soft limit** of 1024KB file size for user `jane`.

<details>
<summary>Solution</summary>

Edit `/etc/security/limits.conf`:
```plaintext
john    hard    nproc    30
jane    soft    fsize    1024
```
</details>

---

### **Scenario: Kernel Parameter (`vm.swappiness`)**
**Task**:  
Set `vm.swappiness=10` persistently and apply immediately.

<details>
<summary>Solution</summary>

```bash
# Apply immediately
sudo sysctl vm.swappiness=10

# Persistent change
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swappiness.conf
```
</details>

---

### **Scenario: LVM Configuration**
**Task**:  
1. Create PVs from `/dev/vdc` and `/dev/vdd`.  
2. Create VG `volume1` and LV `website_files` (3GB).

<details>
<summary>Solution</summary>

```bash
sudo pvcreate /dev/vdc /dev/vdd
sudo vgcreate volume1 /dev/vdc /dev/vdd
sudo lvcreate -n website_files -L 3G volume1
```
</details>

---

### **Scenario: Git Repository Setup**
**Task**:  
1. Initialize `~/kode` as Git repo.  
2. Link to remote `https://github.com/kodekloudhub/git-for-beginners-course.git`.  
3. Pull `master` branch.

<details>
<summary>Solution</summary>

```bash
cd ~/kode
git init
git remote add origin https://github.com/kodekloudhub/git-for-beginners-course.git
git pull origin master
```
</details>

---

### **Scenario: NFS Server Configuration**
**Task**:  
Share `/home` as read-only for `10.0.0.0/24`.

<details>
<summary>Solution</summary>

Edit `/etc/exports`:
```plaintext
/home 10.0.0.0/24(ro,sync)
```

```bash
sudo exportfs -a  # Apply changes
```
</details>

---

### **Scenario: Disk Partitioning**
**Task**:  
1. Create two 500M partitions on `/dev/vdb`.  
2. Create an ext4 filesystem on the first partition and an xfs filesystem on the second partition.  
3. Create two directories: `/part1` and `/part2`.  
4. Manually mount the filesystems and configure auto-mount at boot.

<details>
<summary>Solution</summary>

```bash
# Create partitions
sudo fdisk /dev/vdb

# Create filesystems
sudo mkfs.ext4 /dev/vdb1
sudo mkfs.xfs /dev/vdb2

# Create mount points
sudo mkdir /part1 /part2

# Mount filesystems
sudo mount /dev/vdb1 /part1
sudo mount /dev/vdb2 /part2

# Configure auto-mount
echo "/dev/vdb1 /part1 ext4 defaults 0 2" | sudo tee -a /etc/fstab
echo "/dev/vdb2 /part2 xfs defaults 0 2" | sudo tee -a /etc/fstab
```
</details>

---

### **Scenario: Swap File Configuration**
**Task**:  
Create a 1024MB swap file `/swfile` and enable it persistently.

<details>
<summary>Solution</summary>

```bash
sudo fallocate -l 1G /swfile
sudo chmod 600 /swfile
sudo mkswap /swfile
sudo swapon /swfile

# Persistent configuration
echo "/swfile none swap sw 0 0" | sudo tee -a /etc/fstab
```
</details>

---

### **Scenario: SSH Configuration**
**Task**:  
Disable X11 globally but enable it for user `bob`.

<details>
<summary>Solution</summary>

Edit `/etc/ssh/sshd_config`:
```plaintext
X11Forwarding no

Match User bob
    X11Forwarding yes
```
</details>

---

### **Scenario: Filesystem Cleanup**
**Task**:  
Find the directory with the largest file under `/data` and delete that file.

<details>
<summary>Solution</summary>

```bash
sudo find /data -type f -exec du -h {} + | sort -rh | head -1 | awk '{print $2}' | xargs sudo rm -f
```
</details>

---

### **Scenario: Docker Operations**
**Task**:  
1. Remove the `nginx` image.  
2. Run an `httpd` container named `apache_container` with a restart policy.

<details>
<summary>Solution</summary>

```bash
# Remove nginx image
docker rmi nginx

# Run httpd container
docker run -d --name apache_container -p 80:80 --restart on-failure:3 httpd
```
</details>

---

### **Scenario: ACL Configuration**
**Task**:  
Allow `janet` to read/write `/opt/aclfile`.

<details>
<summary>Solution</summary>

```bash
sudo setfacl -m u:janet:rw /opt/aclfile
```
</details>

---

### **Scenario: SELinux Configuration**
**Task**:  
1. Check SELinux mode and save it to `/opt/selinuxmode.txt`.  
2. Restore the default label for `/usr/bin/less`.

<details>
<summary>Solution</summary>

```bash
# Check SELinux mode
sestatus | grep 'Current mode' | awk '{print $3}' | sudo tee /opt/selinuxmode.txt

# Restore context
sudo restorecon -v /usr/bin/less
```
</details>

---

### **Scenario: LVM Resize**
**Task**:  
Add `/dev/vdc` to `volume1` and resize `lv1` to 2GB.

<details>
<summary>Solution</summary>

```bash
sudo vgextend volume1 /dev/vdc
sudo lvextend -L 2G /dev/volume1/lv1
sudo resize2fs /dev/volume1/lv1  # If using ext4
```
</details>
