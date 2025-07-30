# Disk Troubleshooting Guide for RHEL Systems
This document outlines common disk-related problems you might encounter on Red Hat Enterprise Linux (RHEL) 
servers, along with practical steps to identify and resolve them. ---

## Table of Contents
1) Disk Not Detected 
2) Partition Table Issues 
3) Mount Failures 
4) Disk Full or Out of Inodes 
5) Filesystem Corruption 
6) LVM Issues 
7) Disk I/O Performance Degradation 
8) Filesystem Resize Problems 
9) Permissions and Ownership Issues 
10) Boot Failures Due to Disk Problems 

### 1) Disk Not Detected
```bash

# Scan for new disks manually
echo "- - -" > /sys/class/scsi_host/host0/scan

# Or rescan all SCSI hosts
for host in /sys/class/scsi_host/host*; do
    echo "- - -" > "$host/scan" done

# Check visibility
lsblk fdisk -l 
``` 

### 2) Partition Table Issues
```bash

# View partition info
fdisk -l /dev/sdX

# Create a GPT partition table
parted /dev/sdX mklabel gpt

# Create MBR instead if needed
parted /dev/sdX mklabel msdos

# Create new partition
fdisk /dev/sdX 
``` 

### 3) Mount Failures
```bash

# Check /etc/fstab
blkid mount -a

# Check logs
journalctl -xe dmesg | grep -i mount

# Try manual mount
mount /dev/sdX1 /mnt 
``` 

### 4) Disk Full or Out of Inodes
```bash

# Check usage
df -h df -i

# Find large directories
du -sh /* 2>/dev/null | sort -h

# Find recent large files
find / -mtime -1 -ls

# Rotate logs and remove old files
logrotate --force /etc/logrotate.conf rm -rf /var/log/*.gz 
``` 

### 5) Filesystem Corruption
```bash

# Run fsck (only if unmounted)
umount /dev/sdX1 fsck -f /dev/sdX1 
``` 

### 6) LVM Issues
```bash

# Scan and activate
vgscan lvscan vgchange -ay

# Mount LV
mount /dev/mapper/vgname-lvname /mnt

# Check PV/VG/LV
pvs vgs lvs 
``` 

### 7) Disk I/O Performance Degradation
```bash

# Monitor I/O
iostat -xm 5 iotop dstat -dny

# Check SMART
smartctl -a /dev/sdX

# Check and change scheduler
cat /sys/block/sdX/queue/scheduler echo mq-deadline > /sys/block/sdX/queue/scheduler 
``` 

### 8) Filesystem Resize Problems
```bash

# ext4
resize2fs /dev/mapper/vgname-lvname

# xfs
xfs_growfs /mountpoint 
``` 

### 9) Permissions and Ownership Issues
```bash

# Adjust permissions
chown user:group /mountpoint chmod 755 /mountpoint

# Verify
ls -l /mountpoint 
``` 

### 10) Boot Failures Due to Disk Problems
```bash

# Rescue steps
mount -o remount,rw /

# Edit fstab
nano /etc/fstab

# Reboot system
reboot 
``` 

### Common Tools
```bash 
lsblk 	# Block devices blkid 
	# UUIDs and types fdisk / parted 
	# Partitioning 
df -h / du -sh 
# Usage 
fsck 
# Repair FS lvs/vgs/pvs 
# LVM tools iostat/iotop 
# Performance smartctl 
# Disk health 
``` 

### Best Practices
Use UUIDs in `/etc/fstab` - Monitor disk usage and alert above 80% - Clean logs regularly - Use LVM where 
possible - Automate regular backups
