# Linux System Operations Guide

## 1. System Management

### Boot, Reboot, and Shutdown
- **Reboot**:  
  ```bash
  sudo systemctl reboot
  ```
- **Shutdown**:  
  ```bash
  sudo systemctl poweroff
  ```
- **Force Reboot/Shutdown** (Use sparingly):  
  ```bash
  sudo systemctl reboot --force --force  # Like a hardware reset
  sudo systemctl poweroff --force --force  # Like unplugging power
  ```

### Scheduled Tasks
- **Shutdown at 02:00 AM**:  
  ```bash
  sudo shutdown 02:00
  ```
- **Reboot in 15 minutes**:  
  ```bash
  sudo shutdown -r +15
  ```
- **Add a warning message**:  
  ```bash
  sudo shutdown -r +1 'Scheduled kernel upgrade'
  ```

---

## 2. Scripting and Automation

### Basic Script Creation
1. **Create a script**:  
   ```bash
   touch script.sh && chmod +x script.sh
   ```
2. **Add a shebang**:  
   ```bash
   #!/bin/bash
   date >> /tmp/script.log
   cat /proc/version >> /tmp/script.log
   ```
3. **Run the script**:  
   ```bash
   ./script.sh
   ```

### Advanced Script Logic
- **Backup with versioning**:  
  ```bash
  #!/bin/bash
  if test -f /tmp/archive.tar.gz; then
    mv /tmp/archive.tar.gz /tmp/archive.tar.gz.OLD
  fi
  tar acf /tmp/archive.tar.gz /etc/apt/
  ```

### Cron Jobs
- **Edit cron table**:  
  ```bash
  crontab -e
  ```
- **Run a job daily at 6:35 AM**:  
  ```text
  35 6 * * * /usr/bin/touch /home/user/test_file
  ```
- **List cron jobs**:  
  ```bash
  crontab -l
  ```

---

## 3. Managing Services with systemd

### Service Commands
- **Check status**:  
  ```bash
  systemctl status ssh.service
  ```
- **Start/Stop/Restart**:  
  ```bash
  sudo systemctl start ssh.service
  sudo systemctl stop ssh.service
  sudo systemctl restart ssh.service
  ```
- **Enable at boot**:  
  ```bash
  sudo systemctl enable ssh.service
  ```

### Masking Services
- **Prevent a service from starting**:  
  ```bash
  sudo systemctl mask atd.service
  ```
- **Unmask**:  
  ```bash
  sudo systemctl unmask atd.service
  ```

---

## 4. Log Management

### Key Log Files
- **Authentication logs**:  
  ```bash
  less /var/log/auth.log
  ```
- **System logs**:  
  ```bash
  journalctl -u ssh.service  # Logs for SSH service
  ```

### Live Log Monitoring
- **Follow logs in real-time**:  
  ```bash
  journalctl -f  # Follow all logs
  tail -F /var/log/auth.log  # Follow auth.log
  ```

---

## 5. Resource Monitoring

### Disk and Memory
- **Disk usage**:  
  ```bash
  df -h  # Human-readable format
  du -sh /var/log  # Check directory size
  ```
- **Memory usage**:  
  ```bash
  free -h
  ```

### CPU Load
- **Check load averages**:  
  ```bash
  uptime
  ```
  Output:  
  `17:24:55 up 32 min, 1 user, load average: 0.05, 0.05, 0.01`

---

## 6. Security with SELinux

### Context Management
- **View file contexts**:  
  ```bash
  ls -Z /usr/sbin/sshd
  ```
  Output:  
  `system_u:object_r:sshd_exec_t:s0`
- **View process contexts**:  
  ```bash
  ps axZ | grep sshd
  ```

### SELinux Modes
- **Check enforcement mode**:  
  ```bash
  getenforce  # Output: Enforcing, Permissive, or Disabled
  ```

---

## 7. Docker Basics

### Container Management
- **Run an Nginx container**:  
  ```bash
  sudo docker run -d -p 80:80 nginx
  ```
- **List containers**:  
  ```bash
  sudo docker ps
  ```
- **Resolve permissions**:  
  ```bash
  sudo usermod -aG docker $USER  # Add user to docker group
  ```

---