# **Storage and Virtualization**


## **1. Virtualization Basics**
- **Virtual Machines (VMs)**  
  - Emulate physical computers using hypervisors (e.g., QEMU-KVM).  
  - **QEMU**: Quick Emulator for hardware virtualization.  
  - **KVM**: Kernel-based Virtual Machine for Linux, enabling direct hardware access.  
  - **VIRSH**: CLI tool to manage VMs (e.g., `virsh list`, `virsh start`).  
- **Cloud Providers**: AWS, Azure, Google Cloud, Digital Ocean offer scalable VM hosting.  
- **Commands**: 
  - The simplest invocation to interactively install a Fedora 29 KVM VM with recommended defaults. virt-viewer(1) will be launched to graphically interact with the VM install
    ```bash
    sudo virt-install --install fedora29
    ```
  - Similar, but use libosinfo's unattended install support, which will perform the fedora29 install automatically without user intervention:
    ```bash
    sudo virt-install --install fedora29 --unattended
    ```
  - Install a Windows 10 VM, using 40GiB storage in the default location and 4096MiB of ram, and ensure we are connecting to the system libvirtd instance:
    ```bash
    virt-install \
      --connect qemu:///system \
      --name my-win10-vm \
      --memory 4096 \
      --disk size=40 \
      --osinfo win10 \
      --cdrom /path/to/my/win10.iso
      ```

  - Install a CentOS 7 KVM from a URL, with recommended device defaults and default required storage, but specifically request VNC graphics instead of the default SPICE, and request 8 virtual CPUs and 8192 MiB of memory:
    ```bash
    virt-install \
                --connect qemu:///system \
                --memory 8192 \
                --vcpus 8 \
                --graphics vnc \
                --osinfo centos7.0 \
                --location http://mirror.centos.org/centos-7/7/os/x86_64/
    ```
  - Create a VM around an existing debian9 disk image:
    ```bash
    virt-install \
                --import \
                --memory 512 \
                --disk /home/user/VMs/my-debian9.img \
                --osinfo debian9
    ```
---

## **2. Swap Space Management**
- **Purpose**: Extend RAM by temporarily moving inactive data to disk.  
- **Commands**:  
  - `swapon --show`: Check active swap areas.  
  - `mkswap /dev/vdb3`: Format a partition as swap.  
  - `swapon /dev/vdb3`: Enable swap partition.  
  - `swapoff /dev/vdb3`: Disable swap.  
- **Swap Files**:  
  - Create with `dd if=/dev/zero of=/swap bs=1M count=2048`.  
  - Set permissions: `chmod 600 /swap`.  
  - Activate: `mkswap /swap && swapon /swap`.  
- **Persistent Configuration**:  
  - Add to `/etc/fstab`:  
    ```bash
    /dev/vdb3 none swap defaults 0 0
    # or for swap files:
    /swap none swap defaults 0 0
    ```

---

## **3. Filesystems and Mounting**
- **Partitioning**:  
  - **NTFS**: Windows filesystem.  
  - **EXT4/XFS**: Linux filesystems.  
- **Mounting Commands**:  
  - `mount /dev/vdb1 /mnt`: Mount a partition.  
  - `umount /mnt`: Unmount.  
- **Auto-Mount at Boot**:  
  - Edit `/etc/fstab`:  
    ```bash
    /dev/vdb1 /mybackups xfs defaults 0 2
    # Using UUID:
    UUID=a51d7731-b833-4c07-bl71 /mybackups xfs defaults 0 2
    ```  
  - **Fields**: `<device> <mount-point> <fs-type> <options> <dump> <fsck>`.  

---

## **4. Filesystem Options**
- **Common Mount Options**:  
  - `rw` (read-write), `ro` (read-only), `noexec` (block executables), `nosuid` (disable SUID).  
- **Adjust Options**:  
  - Remount: `mount -o remount,rw /dev/vdb2 /mnt`.  
- **XFS-Specific Options**:  
  - E.g., `allocsize=32K` for I/O optimization.  

---

## **5. Network File System (NFS)**
- **Server Setup**:  
  1. Install `nfs-kernel-server`.  
  2. Edit `/etc/exports`:  
     ```bash
     /shared/dir client-IP(rw,sync,no_subtree_check)
     ```  
  3. Apply: `exportfs -r`.  
- **Client Setup**:  
  - Install `nfs-common`.  
  - Mount: `mount server-IP:/shared/dir /mnt`.  
  - Auto-mount via `/etc/fstab`:  
    ```bash
    server-IP:/shared/dir /mnt nfs defaults 0 0
    ```  

---

## **6. Storage Monitoring**
- **Tools**:  
  - `iostat`: Reports disk I/O statistics (e.g., `kB_read/s`, `tps`).  
  - `pidstat`: Monitors per-process I/O usage.  
- **Key Metrics**:  
  - **High `tps`**: Frequent read/write operations.  
  - **High `kB_read/s` or `kB_wrtn/s`**: Large data transfers.  

---

## **7. Network Block Devices (NBD)**
- **Concept**: Remote block devices accessed as local drives (e.g., `/dev/nbd0`).  
- **Use Case**: Attach remote storage (e.g., from Server 2’s `/dev/vdb` to Client’s `/dev/nbd0`).  

---

## **8. Logical Volume Manager (LVM)**

- **Concept**: LVM allows flexible disk management by abstracting physical storage into logical volumes. It enables resizing, snapshots, and combining multiple physical disks into a single logical unit.
- **Key Components**:
  - **Physical Volume (PV)**: Underlying physical storage (e.g., `/dev/sda1`).
  - **Volume Group (VG)**: Combines multiple PVs into a storage pool.
  - **Logical Volume (LV)**: Allocated storage from a VG, used as a mountable filesystem.

### **Useful Commands**

#### **Physical Volume Management**
- Create a PV: `pvcreate /dev/sdX`
- Display PVs: `pvs` or `pvdisplay`
- Remove a PV: `pvremove /dev/sdX`

#### **Volume Group Management**
- Create a VG: `vgcreate my_vg /dev/sdX`
- Display VGs: `vgs` or `vgdisplay`
- Extend a VG: `vgextend my_vg /dev/sdY`
- Reduce a VG: `vgreduce my_vg /dev/sdY`
- Remove a VG: `vgremove my_vg`

#### **Logical Volume Management**
- Create an LV: `lvcreate -L 10G -n my_lv my_vg`
- Display LVs: `lvs` or `lvdisplay`
- Extend an LV: `lvextend -L +5G /dev/my_vg/my_lv`
- Resize filesystem after extending:
  - For EXT4: `resize2fs /dev/my_vg/my_lv`
  - For XFS: `xfs_growfs /mount/point`
- Reduce an LV (requires unmounting):
  1. Resize filesystem: `resize2fs /dev/my_vg/my_lv 5G`
  2. Reduce LV: `lvreduce -L 5G /dev/my_vg/my_lv`
- Remove an LV: `lvremove /dev/my_vg/my_lv`

#### **Snapshots**
- Create a snapshot: `lvcreate -L 1G -s -n my_lv_snap /dev/my_vg/my_lv`
- Restore from a snapshot: `lvconvert --merge /dev/my_vg/my_lv_snap`

#### **Monitoring and Troubleshooting**
- Check LVM status: `lvscan`, `vgscan`, `pvscan`
- Repair a VG: `vgck my_vg`
- Activate a VG: `vgchange -ay my_vg`

### **Example Workflow**
1. Create a PV: `pvcreate /dev/sdX`
2. Create a VG: `vgcreate my_vg /dev/sdX`
3. Create an LV: `lvcreate -L 10G -n my_lv my_vg`
4. Format the LV: `mkfs.ext4 /dev/my_vg/my_lv`
5. Mount the LV: `mount /dev/my_vg/my_lv /mnt`
6. Add to `/etc/fstab` for persistence:
   ```bash
   /dev/my_vg/my_lv /mnt ext4 defaults 0 0
   ```

---

### **Summary of Commands**
| **Task**               | **Command**                          |
|-------------------------|--------------------------------------|
| Check swap usage        | `swapon --show`                      |
| Format swap partition   | `mkswap /dev/vdb3`                   |
| Mount filesystem        | `mount /dev/vdb1 /mnt`               |
| Edit fstab              | `sudo vim /etc/fstab`                |
| NFS server setup        | `sudo apt install nfs-kernel-server` |
| Monitor disk I/O        | `iostat` or `pidstat -d`             |
---
