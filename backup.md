Backup Troubleshooting (RHEL)

1) Check Disk Space
```bash
   df -h                                # Check overall disk usage
   du -sh /backups/*                    # Check size of existing backup files
```
2) Verify Backup Script
```bash
   ls -l /usr/local/bin/backup.sh       # Confirm script exists
   chmod +x /usr/local/bin/backup.sh    # Make script executable
```
3) Run Backup Manually and Check Status
```bash
   sudo /usr/local/bin/backup.sh        # Run backup manually
   echo $?                              # 0 = success, non-zero = error
```
4) Check Cron Job for Automation
```bash
   crontab -l                           # Show current user's crontab
   sudo crontab -l -u <username>        # Check cron for specific user
```
   Sample cron entry (runs daily at 2 AM):
   0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

5) Check Backup Destination
```bash
   ls -lh /backups                      # List backup files with size info
   ls -lh /backups/backup_$(date +%F).tar.gz   # Verify today’s backup
```
6) Check Log File (if logging enabled)
```bash
   tail -n 50 /var/log/backup.log       # Check for errors or confirmation
```
7) Check SELinux (if enabled)
```bash
   getenforce                           # Check if SELinux is enforcing
   sudo setenforce 0                    # Temporarily disable for testing
```
   #Relabel if necessary:
   restorecon -Rv /backups

8) Check System Logs for Errors
```bash
   sudo journalctl -xe | grep backup    # Look for backup-related entries
   sudo dmesg | grep tar                # Check kernel logs for tar issues
```
9) Test Restore Process
```bash
   tar -tzf /backups/backup_<date>.tar.gz        # List contents of archive
   tar -xzf /backups/backup_<date>.tar.gz -C /tmp/test_restore
```
10) Remote Backup Sync (if configured)
```bash
    rsync -avz /backups/ user@remote:/remote/dir     # Copy backups to remote host
```
#If connection issues:
```bash
    ping <remote_ip>
    ssh user@<remote_ip>
```
