# Linux User and Group Management Guide 

## **1. User Management**

### **Commands**
#### **`adduser` (Debian/Ubuntu)**
- **Description**: Interactive tool to create users with home directories and prompts for details.
- **Options**:
  ```bash
  adduser --uid 1005 username  # Assign specific UID
  adduser --system username    # Create system user (no home dir)
  adduser --disabled-login     # Create user without password
  ```
- **File Created**: `/etc/passwd`, `/etc/shadow`, `/home/username`.

#### **`useradd` (Universal)**
- **Description**: Low-level command to create users (defaults in `/etc/default/useradd`).
- **Options**:
  ```bash
  useradd -m -d /home/customdir username  # Create home dir
  useradd -s /sbin/nologin username      # Set default shell
  useradd -u 1500 username                # Assign UID
  ```

#### **`passwd`**
- **Description**: Set/change passwords or lock accounts.
- **Options**:
  ```bash
  passwd username           # Set password
  passwd -l username        # Lock account
  passwd -e username        # Expire password (force change on login)
  ```

#### **`deluser` (Debian/Ubuntu) / `userdel` (RHEL)**
- **Description**: Remove users.
  ```bash
  deluser --remove-home username  # Delete user + home dir
  userdel -r username             # Same in RHEL
  ```

#### **`usermod`**
- **Description**: Modify user properties.
- **Options**:
  ```bash
  usermod -aG sudo username   # Add to sudo group
  usermod -d /newhome username  # Change home dir
  usermod -L username         # Lock account
  ```

#### **`chage`**
- **Description**: Manage password aging policies.
  ```bash
  chage -l username           # List policies
  chage -E 2024-12-31 username  # Set account expiry
  ```

---

## **2. Group Management**

### **Commands**
#### **`groupadd`**
- **Description**: Create new groups.
  ```bash
  groupadd -g 2000 devteam  # Assign GID 2000
  ```

#### **`groupdel`**
- **Description**: Delete groups (group must be empty).
  ```bash
  groupdel devteam
  ```

#### **`gpasswd`**
- **Description**: Manage group passwords/members.
  ```bash
  gpasswd -a user devteam  # Add user to group
  gpasswd -d user devteam  # Remove user from group
  ```

#### **`groups`**
- **Description**: List groups a user belongs to.
  ```bash
  groups username  # Output: username : sudo devteam
  ```

#### **`usermod -g`**
- **Description**: Change primary group.
  ```bash
  usermod -g devteam username
  ```

---

## **3. Key Files**

### **`/etc/passwd`**
- **Structure**:  
  `username:x:UID:GID:FullName:/home/username:/bin/bash`
- **View**: `cat /etc/passwd | grep username`

### **`/etc/shadow`**
- **Purpose**: Stores password hashes and expiry policies.
- **Permissions**: Accessible only by root.

### **`/etc/group`**
- **Structure**:  
  `groupname:x:GID:user1,user2`

### **`/etc/sudoers`**
- **Edit Safely**: `sudo visudo`
- **Example Entry**:  
  `username ALL=(ALL) NOPASSWD:ALL`

### **`/etc/security/limits.conf`**
- **Purpose**: Set user/group resource limits (e.g., max processes).
- **Example**:  
  `username hard nproc 500`

---

## **4. Environment & Shell**

### **Commands**
- **`printenv`**: List environment variables.
- **`login` / `logout`**: Start/end shell sessions.
- **`lastlog`**: Show recent user logins.

### **Files**
- **`/etc/environment`**: System-wide environment variables.
- **`/etc/profile.d/`**: Scripts run at user login.

---

## **5. Advanced Tools**

### **`ulimit`**
- **Description**: Control user resource limits.
  ```bash
  ulimit -a            # Show all limits
  ulimit -u 500        # Set max user processes
  ```

### **`sudo -iu`**
- **Description**: Login as another user (simulates full login).
  ```bash
  sudo -iu username
  ```

### **`stat`**
- **Description**: View file/directory metadata (UID/GID).
  ```bash
  stat /home/username
  ```

---

## **6. LFCS Exam Essentials**

### **Key Tasks to Master**
1. **Create/Delete Users with Custom UID/GID**:
   ```bash
   useradd -u 1500 -g devteam -s /bin/bash john
   ```
2. **Set Password Policies**:
   ```bash
   chage -M 90 -W 7 john  # Expire in 90 days, warn 7 days before
   ```
3. **Manage Groups**:
   ```bash
   groupmod -n newgroup oldgroup  # Rename group
   ```
4. **Troubleshoot Permissions**:
   ```bash
   ls -ln /home  # Check numeric UID/GID
   id username   # Verify group membership
   ```

### **Critical Files**
- **`/etc/passwd`**: User accounts.
- **`/etc/shadow`**: Password hashes.
- **`/etc/group`**: Group definitions.
- **`/etc/sudoers`**: Sudo privileges.

---

## **7. Quick Reference Table**

| Command          | Purpose                              | Example                          |
|------------------|--------------------------------------|----------------------------------|
| `useradd -m`     | Create user with home dir            | `useradd -m alice`              |
| `usermod -aG`    | Add user to secondary group          | `usermod -aG sudo alice`        |
| `gpasswd -a`     | Add user to group                    | `gpasswd -a alice dev`          |
| `chage -l`       | List password expiry                 | `chage -l alice`                |
| `ls -ln`         | View numeric UID/GID                 | `ls -ln /home`                  |

---

## 8. Lets do some scenario-based practice questions for LFCS exam preparation


### **Scenario 1: User Creation**
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
**Verification**:  
- Check `/etc/passwd` for UID 3000  
- Confirm shell in `grep backupadmin /etc/passwd`
</details>

---

### **Scenario 2: Password Policy**
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
**Key Files**:  
- Changes reflected in `/etc/shadow`
</details>

---

### **Scenario 3: Group Management**
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
**Note**: Members of "sysadmin" can now use `gpasswd -a user developers`
</details>

---

### **Scenario 4: Troubleshooting Permissions**
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
**Key Command**:  
`ls -ld /var/log/audit` to verify group ownership
</details>

---

### **Scenario 5: Sudo Access**
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
**Verification**:  
```bash
sudo -l -U devuser
```
</details>

---

### **Scenario 6: Resource Limits**
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
**Verify**:  
```bash
su - testuser
ulimit -a
```
</details>

---

### **Scenario 7: System User**
**Task**:  
Create a system user "nginx" without home directory for running web server

<details>
<summary>Solution</summary>

```bash
sudo useradd -r -s /sbin/nologin nginx

# Verify
grep nginx /etc/passwd
```
**Flags**:  
`-r` = system user, `-s` = set login shell
</details>

---
## **8. Exam Tips**
- **Practice**: Simulate exam tasks in a VM (e.g., create users with specific UIDs).
- **Know Files**: Memorize structure of `/etc/passwd`, `/etc/group`.
- **Use Man Pages**: `man useradd`, `man groupmod` during the exam.
- **SELinux/AppArmor**: Basic awareness (e.g., restore context with `restorecon`).
- Always verify changes with `id`, `ls -l`, or relevant config files. 
- Use `-aG` with `usermod` to preserve existing group memberships.
- Remember `visudo` is safer than directly editing `/etc/sudoers`.
- For resource limits, changes apply to new sessions only

