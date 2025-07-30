# Disk Troubleshooting Guide for RHEL Systems
## Table of Contents
1. Disk Not Detected 2. Partition Table Issues 3. Mount Failures 4. Disk Full or Out of Inodes 5. Filesystem 
Corruption 6. LVM Issues 7. Disk I/O Performance Degradation 8. Filesystem Resize Problems 9. Permissions and 
Ownership Issues 10. Boot Failures Due to Disk Problems ---
## 1. Disk Not Detected
**Problem:** Newly attached disk is not visible to the operating system. **Solution:** ```bash
# Scan for new disks manually
echo "- - -" > /sys/class/scsi_host/host0/scan
# Or rescan all SCSI hosts in a loop
for host in /sys/class/scsi_host/host*; do
    echo "- - -" > "$host/scan" done
# Check disk visibility
lsblk fdisk -l ``` ---
## 2. Partition Table Issues
**Problem:** Disk shows up, but `fdisk` reports errors or unknown partition table. **Solution:** ```bash
# Display current partition information
fdisk -l /dev/sdX
# Create a new GPT partition table
parted /dev/sdX mklabel gpt
# Or use MBR if needed
parted /dev/sdX mklabel msdos
# Create new partitions interactively
fdisk /dev/sdX ``` ---
## 3. Mount Failures
**Problem:** Disk doesn't mount manually or during system boot. **Solution:** ```bash
# Check fstab and device UUIDs
blkid mount -a
# Review logs for mount errors
journalctl -xe dmesg | grep -i mount
# Try mounting directly
mount /dev/sdX1 /mnt ``` ---
## 4. Disk Full or Out of Inodes
**Problem:** The disk appears full, or the system reports no space left even though it's not obvious why. 
**Solution:** ```bash
# Check disk and inode usage
df -h df -i
# Identify large files or directories
du -sh /* 2>/dev/null | sort -h
# Find recent file modifications
find / -mtime -1 -ls
# Force log rotation or clean old logs (with caution)
logrotate --force /etc/logrotate.conf rm -rf /var/log/*.gz ``` ---
## 5. Filesystem Corruption
**Problem:** Filesystem is marked as dirty or inaccessible due to errors. **Solution:** ```bash
# Only run fsck on unmounted filesystems
umount /dev/sdX1 fsck -f /dev/sdX1
# For root filesystems, use a rescue environment or recovery mode
``` ---
## 6. LVM Issues
**Problem:** Logical volumes are missing, inactive, or not mounting. **Solution:** ```bash
# Scan and activate volumes
vgscan lvscan vgchange -ay
# Try mounting the logical volume
mount /dev/mapper/vgname-lvname /mnt
# Validate PVs, VGs, and LVs
pvs vgs lvs ``` ---
## 7. Disk I/O Performance Degradation
**Problem:** Disk performance is noticeably slower. **Solution:** ```bash
# Check I/O stats and activity
iostat -xm 5 iotop dstat -dny
# Check drive health using SMART
smartctl -a /dev/sdX
# View and optionally change the I/O scheduler
cat /sys/block/sdX/queue/scheduler echo mq-deadline > /sys/block/sdX/queue/scheduler ``` ---
## 8. Filesystem Resize Problems
**Problem:** You resized a disk or volume, but the filesystem doesn't reflect the new space. **Solution:** 
For ext4 filesystems: ```bash resize2fs /dev/mapper/vgname-lvname ``` For XFS filesystems: ```bash xfs_growfs 
/mountpoint ``` ---
## 9. Permissions and Ownership Issues
**Problem:** Users cannot read or write to the mounted disk. **Solution:** ```bash
# Adjust ownership and permissions
chown user:group /mountpoint chmod 755 /mountpoint
# Review permissions using ls
ls -l /mountpoint ``` ---
## 10. Boot Failures Due to Disk Problems
**Problem:** The system fails to boot because of incorrect entries in `/etc/fstab` or missing disks. 
**Solution:** ```bash
# Boot into rescue mode or use recovery shell Remount the root filesystem in read-write mode
mount -o remount,rw /
# Comment out or fix invalid fstab entries
nano /etc/fstab
# Reboot after fixing configuration
``` ---
## Common Tools and Commands
| Command | Description | ---------------|-------------------------------------| `lsblk` | Lists block 
| devices | `blkid` | Shows UUID and filesystem types | `fdisk`, `parted` | Disk partitioning utilities | 
| `df`, `du` | Disk usage reporting | `fsck` | Filesystem check and repair | `lvs`, `vgs`, `pvs` | LVM 
| management tools | `iostat`, `iotop`, `dstat` | I/O monitoring tools | `smartctl` | Disk health monitoring 
| (SMART) |
---
## Best Practices
- Always use UUIDs in `/etc/fstab` to avoid device name issues. - Set up disk usage monitoring and alerts. - 
Use LVM for flexible storage and snapshots. - Clean up unnecessary logs and temp files periodically. - 
Perform regular backups of critical filesystems.
