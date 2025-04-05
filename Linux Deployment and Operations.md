## Linux Operations & Deployment Notes 

### **1. Boot, Shutdown & System Initialization**
#### Key Commands:
```bash
# Shutdown/Reboot
sudo shutdown -h now           # Halt immediately
sudo shutdown -r +5 "Rebooting in 5 minutes"  # Custom message
sudo systemctl reboot --force  # Force reboot (avoid unless necessary)

# Systemd Targets
sudo systemctl get-default     # Check current default target
sudo systemctl set-default multi-user.target  # CLI-only boot
sudo systemctl isolate rescue.target          # Rescue mode (root shell)

# GRUB & Boot Repair
sudo grub2-mkconfig -o /boot/grub2/grub.cfg   # Rebuild GRUB config
sudo chroot /mnt/sysroot                      # Repair OS from live environment
sudo dnf reinstall grub2-efi grub2-efi-modules # Fix bootloader
```

---

### **2. Systemd Services & Process Management**
#### Service Control:
```bash
# Service Lifecycle
sudo systemctl edit --full sshd.service   # Edit unit file
sudo systemctl daemon-reload              # Reload after editing
sudo systemctl mask atd.service           # Prevent service from starting

# Process Signals & Priorities
pgrep -a nginx           # Find PID of nginx processes
kill -SIGHUP <PID>       # Gracefully reload a process
renice -n 10 -p <PID>    # Change process priority (Niceness)
```

#### Process Analysis:
```bash
# Advanced Tools
lsof -p <PID>            # List open files by a process
ps axZ | grep httpd      # View SELinux context of processes
top -c                   # Monitor processes with command-line details
```

---

### **3. Logging & Auditing**
#### Journalctl & Log Files:
```bash
# Filter Logs by Priority/Time
journalctl -p err -S "2024-03-01" --until "2024-03-02"  # Errors in a date range
journalctl -u nginx.service --since "10 minutes ago"    # Service-specific logs
journalctl -k                          # Kernel logs only

# Syslog & File-based Logs
grep -r "authentication failure" /var/log/  # Search auth logs
tail -F /var/log/secure                     # Follow live security logs
```

---

### **4. Task Scheduling**
#### Cron/Anacron/At:
```bash
# Cron Examples
0 4 * * * /opt/backup.sh         # Daily at 4 AM
*/15 * * * * /opt/monitor.sh     # Every 15 minutes

# Anacron Configuration (for missed jobs)
sudo nano /etc/anacrontab        # Example entry:
@weekly 15 cron.weekly  run-parts /etc/cron.weekly

# One-time Tasks with `at`
echo "/opt/cleanup.sh" | at 02:00  # Schedule at 2 AM
atq                                # List pending jobs
```

---

### **5. Package Management**
#### Advanced Tools (RHEL/Debian):
```bash
# Query Packages
dnf repoquery --list nginx        # List files in a package
rpm -ql nginx                     # Same as above (RHEL)
dpkg -L nginx                     # Debian equivalent

# Dependency Debugging
dnf provides /usr/sbin/nginx      # Find which package provides a file
apt-cache search "Apache HTTP"    # Search package descriptions
```

---

### **6. SELinux & Security**
#### Context Management:
```bash
# Label Files/Processes
ls -Z /var/www/html               # View SELinux context
chcon -t httpd_sys_content_t /var/www/html/index.html  # Change type
restorecon -Rv /var/www/html      # Reset to default context

# User/Role Mapping
sudo semanage login -l            # List SELinux user mappings
sudo semanage user -a -R "staff_r" developer_u  # Create new SELinux user
```

---

### **7. Kernel & Runtime Parameters**
#### sysctl & Tuning:
```bash
# Temporary vs Persistent Changes
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1  # Disable IPv6 (temp)
echo "net.ipv6.conf.all.disable_ipv6=1" >> /etc/sysctl.conf  # Permanent
sysctl -p /etc/sysctl.d/swap.conf                # Load custom config

# Filesystem Checks
sudo xfs_repair /dev/vdb1        # Repair XFS filesystem
sudo fsck.ext4 -p /dev/sda2      # Auto-repair ext4 errors
```

---

### **8. Docker & Containerization**
#### Advanced Commands:
```bash
# Debug Containers
docker logs apache_container      # View container logs
docker exec -it apache_container /bin/bash  # Enter running container

# Image Management
docker rmi $(docker images -q)   # Remove all images
docker save nginx > nginx.tar    # Export image
docker load < nginx.tar          # Import image
```

---

### **9. LVM & Storage**
#### Volume Management:
```bash
# Extend Logical Volume
sudo pvcreate /dev/vdc           # Initialize physical volume
sudo vgextend vg1 /dev/vdc       # Add disk to volume group
sudo lvextend -L +5G /dev/vg1/lv1  # Resize LV
sudo resize2fs /dev/vg1/lv1      # Resize filesystem (ext4)
```

---

### **10. Scripting & Automation**
#### Error Handling & Logging:
```bash
#!/bin/bash
# Backup script with error checking
if tar -czf /backup/archive.tar.gz /data; then
    echo "Backup succeeded: $(date)" >> /var/log/backup.log
else
    echo "Backup failed: $(date)" >> /var/log/backup.log
    exit 1
fi
```

---

### **11. Networking & Firewalls**
#### Advanced Configurations:
```bash
# iptables Persistence
sudo iptables-save > /etc/iptables/rules.v4  # Save IPv4 rules
sudo netfilter-persistent reload             # Apply saved rules

# Network Debugging
ss -tuln                                     # List open ports
tcpdump -i eth0 port 80                     # Capture HTTP traffic
```

---

### **12. User & Group Management**
#### Advanced Tasks:
```bash
# Fix Home Directory Permissions
sudo mkdir /home/jane
sudo chown jane:jane /home/jane
sudo usermod -d /home/jane jane   # Update home directory path

# Password Policies
sudo chage -M 90 jane            # Set password expiry (90 days)
sudo pam_tally2 --user=jane      # Check failed login attempts
```

---

### **13. File Integrity & ACLs**
#### Permissions & Access Control:
```bash
# ACL Management
setfacl -m u:janet:rw /opt/aclfile   # Add read-write for user
getfacl /opt/aclfile                 # View ACL entries

# Security Limits
echo "janet hard nproc 100" >> /etc/security/limits.conf  # Process limit
```

---

### **14. Virtualization (KVM)**
#### VM Creation:
```bash
# Create a VM with virt-install
virt-install \
  --name mockexam2 \
  --ram 1024 \
  --vcpus 1 \
  --os-variant ubuntu22.04 \
  --disk path=/var/lib/libvirt/images/ubuntu.img \
  --noautoconsole

virsh autostart mockexam2          # Enable autostart
```

---

### **15. Git & Source Control**
#### Repository Operations:
```bash
# Clone & Build from Source
git clone https://github.com/htop-dev/htop
cd htop
./autogen.sh && ./configure && make
sudo make install                   # Install to /usr/local/bin
```

---

### **16. Kernel Modules & Hardware**
#### Hardware Info:
```bash
lspci -v | grep -i vga             # List GPU details
lsmod | grep nvidia                # Check loaded kernel modules
modinfo ext4                       # Show module information
```

---

### **Key Fixes from the PDF/MD Files**
- **NTP Configuration**:  
  ```bash
  sudo nano /etc/systemd/timesyncd.conf  # Add NTP pools
  [Time]
  NTP=0.europe.pool.ntp.org 1.europe.pool.ntp.org
  
  sudo systemctl restart systemd-timesyncd
  ```

- **Cron for User `john`**:  
  ```bash
  crontab -u john -e
  # Add: 0 4 * * 3 find /home/john/ -type d -empty -delete
  ```

- **Network Interface Identification**:  
  ```bash
  ip -4 addr show | grep 10.5.5.2 | awk '{print $NF}' > /opt/interface.txt
  ```
