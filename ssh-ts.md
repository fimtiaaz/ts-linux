SSH Troubleshooting

1)Check SSH Service on Server
```bash
sudo systemctl status sshd        
```
```bash
sudo systemctl start sshd         # If not active, start it:
sudo systemctl enable sshd
```

2)From the client machine, test basic connectivity:
```bash	
ping <server_ip>                 
telnet <server_ip> 22
```
3)Check for Authentication Issues

Client private key: ~/.ssh/id_rsa

Server: ~/.ssh/authorized_keys under the user’s home

Permission should be like:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

4)Check if user exists on server
```bash    
id <username>

If not found:
```bash
sudo useradd -m -s /bin/bash <username>
sudo passwd <username>
```    
To add the user to a group (e.g., 'developers'):
```bash
sudo usermod -aG developers <username>
```
5)Check /etc/ssh/sshd_config for:
        
PermitRootLogin yes/no
PasswordAuthentication yes
AllowUsers <username>
    
After editing, restart SSH:
```bash        
sudo systemctl restart sshd
```
6)Check Firewall settings (server-side)
```bash    
sudo firewall-cmd --list-all
```
To allow port 22:
```bash
sudo firewall-cmd --add-port=22/tcp --permanent
sudo firewall-cmd --reload
```
7)Check SSH logs on server
```bash    
sudo journalctl -xe | grep sshd
sudo tail -f /var/log/secure
```
8)Optional: SELinux (if enabled)
```bash    
sudo sestatus
```
Temporarily disable for testing:
```bash
sudo setenforce 0
```
9)Permissions on user’s home directory:
```bash
chmod 755 /home/<username>
```
