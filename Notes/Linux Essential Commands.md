# Essential-Commands

## **1. Logging In**
- **Methods**:
  - **Local**: Text-mode (`CTRL + ALT + F2`) or GUI.
  - **Remote**: SSH (Secure Shell) for text-mode, VNC/RDP for GUI.
- **SSH Example**:
  ```bash
  ssh aaron@192.168.0.17
  ```
- **Virtual Terminals**: Accessed via `CTRL + ALT + F2-F6`.

---

## **2. System Documentation**
- **Tools**:
  - `--help`: Quick command reference (e.g., `ls --help`).
  - `man`: Full manual pages (e.g., `man ls`).
  - `apropos`: Search for commands by keyword (e.g., `apropos directory`).
    - Requires updated database: `sudo mandb`.
---

## **3. File/Directory Management**
- **Listing Files**:
  ```bash
  ls -l      # Long format
  ls -a      # Include hidden files
  ls -lh     # Human-readable sizes
  ```
- **File Operations**:
  - **Create**: `touch file.txt`
  - **Copy**: `cp file.txt backup/`
  - **Move/Rename**: `mv file.txt newname.txt`
  - **Delete**: `rm file.txt` or `rm -r directory/`.

- **Permissions**:
  - **Octal Notation**: `chmod 644 file.txt` (rw-r--r--).
  - **Symbolic Notation**: `chmod u+x file.txt` (add execute for owner).
  - **Ownership**:
    ```bash
    chown user:group file.txt
    chgrp group file.txt
    ```
---

## **4. Searching**
- **Find Files**:
  ```bash
  find /var/log -name "*.log"       # By name
  find / -size +10M                 # By size
  find / -mtime -7                  # Modified in last 7 days
  ```
- **Grep for Text**:
  ```bash
  grep "error" /var/log/syslog      # Basic search
  grep -i "error" file.txt          # Case-insensitive
  grep -r "pattern" /etc/           # Recursive search
  ```
---

## **5. Archiving/Compression**
- **Tar**:
  ```bash
  tar -czvf archive.tar.gz /data    # Create compressed archive
  tar -xzvf archive.tar.gz          # Extract
  ```
- **Compression Tools**:
  - `gzip file.txt` → `file.txt.gz`
  - `bzip2 file.txt` → `file.txt.bz2`
  - `xz file.txt` → `file.txt.xz`.
---

## **6. Redirection/Piping**
- **Output Redirection**:
  ```bash
  ls > output.txt          # Overwrite
  ls >> output.txt         # Append
  grep "error" 2> errors.log # Redirect stderr
  ```
- **Piping**:
  ```bash
  cat file.txt | grep "keyword" | sort
  ```
---

## **7. Networking**
- **SSH**:
  ```bash
  ssh user@remote_ip
  ```
- **IP Configuration**:
  ```bash
  ip a      # Show IP addresses
  ```
---

## **8. Version Control (Git Basics)**
- **Key Commands**:
  ```bash
  git init                  # Initialize repository
  git add file.txt          # Stage changes
  git commit -m "Message"   # Commit
  git push origin master    # Push to remote
  git pull                  # Fetch updates
  ```
---

## **9. SSL/TLS Certificates**
- **OpenSSL**:
  ```bash
  openssl req -newkey rsa:2048 -nodes -keyout key.pem -out csr.pem  # Generate CSR
  ```
---

- **Focus Areas**:
  - File permissions (`chmod`, `chown`).
  - Text processing (`grep`, `sed`, `awk`).
  - Process management (`systemctl`, `ps`, `kill`).
  - Networking (`ssh`, `ip`, `netstat`).
---

