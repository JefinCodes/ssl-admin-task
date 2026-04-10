# 🔐 Secure Cloud VM Setup & Hardening (Azure Ubuntu 22.04)

This project demonstrates the complete setup and security hardening of an Ubuntu 22.04 Virtual Machine on Azure, including firewall configuration, intrusion detection, secure user management, backups, web server deployment, database security, and VPN setup.

---

## 📌 Initial Setup

- Created Azure student account  
- Provisioned VM with Ubuntu 22.04  
- Attached DNS:
  ```
  jefincodes.koreacentral.cloudapp.azure.com
  ```

### System Update

```bash
sudo apt update
sudo apt upgrade
```

### 🔄 Unattended Upgrades

- Automatically checks for security updates  
- Downloads and installs updates daily  
- Runs without manual intervention  

---

## 🔐 Enhanced SSH Security

- Created a non-root sudo user  
- Configured SSH key-based authentication  
- Disabled root login:
  ```bash
  PermitRootLogin no
  ```
- Disabled password authentication:
  ```bash
  PasswordAuthentication no
  ```

---

## 🔥 Firewall & Network Security (UFW)

### Reset Rules
```bash
sudo ufw reset
```

### Default Policies
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow Required Ports
```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### Enable Firewall
```bash
sudo ufw enable
```

### Check Status
```bash
sudo ufw status verbose
```

### Logging
```bash
sudo ufw logging on
sudo ufw logging medium
```

### View Logs
```bash
sudo tail -f /var/log/ufw.log
```

---

## 🛡️ Intrusion Detection System (Suricata)

### Install & Start
```bash
sudo apt update
sudo apt install suricata -y
sudo systemctl enable suricata
sudo systemctl start suricata
```

### Check Status
```bash
sudo systemctl status suricata
```

### Logs
```bash
/var/log/suricata/
sudo tail -f /var/log/suricata/fast.log
```

### Update Rules
```bash
sudo suricata-update
sudo systemctl restart suricata
```

---

## 👥 User & Permission Management

### Create Users
```bash
sudo adduser exam_1
sudo adduser exam_2
sudo adduser exam_3
sudo adduser examadmin
sudo adduser examaudit
```

### Assign Roles
```bash
sudo usermod -aG sudo examadmin
sudo groupadd auditgroup
sudo usermod -aG auditgroup examaudit
```

### Grant Audit Access
```bash
sudo chgrp auditgroup /home/exam_1
sudo chgrp auditgroup /home/exam_2
sudo chgrp auditgroup /home/exam_3

sudo chmod 750 /home/exam_1
sudo chmod 750 /home/exam_2
sudo chmod 750 /home/exam_3
```

**Result:**
- `examaudit` can read all exam users' home directories  
- Exam users cannot access each other's data  

---

## 💾 Backup Script (User Data)

### Script
```bash
sudo nano /usr/local/bin/exam_backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/exam_users"
DATE=$(date +%F)

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/exam_backup_$DATE.tar.gz /home/exam_*
```

### Permissions
```bash
sudo chown examadmin:examadmin /usr/local/bin/exam_backup.sh
sudo chmod 700 /usr/local/bin/exam_backup.sh

sudo mkdir -p /var/backups/exam_users
sudo chown examadmin:examadmin /var/backups/exam_users
sudo chmod 700 /var/backups/exam_users
```

### Schedule (2 AM Daily)
```bash
crontab -e
```

```
0 2 * * * /usr/local/bin/exam_backup.sh
```

---

## 🌐 Web Server Deployment (Nginx)

### Setup
```bash
sudo adduser appuser
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

### Run Applications

#### App 1
```bash
wget https://do.edvinbasil.com/ssl/app
wget https://do.edvinbasil.com/ssl/app.sha256.sig

chmod +x app
./app
```

#### App 2
```bash
git clone https://gitlab.com/tellmeY/issslopen.git
cd issslopen
bun install
bun run index.ts
```

---

### Reverse Proxy Configuration

```bash
sudo nano /etc/nginx/sites-available/default
```

```nginx
server {
    listen 80;
    server_name wibblenox.sslnitc.site;

    location /server1/ {
        proxy_pass http://127.0.0.1:8008/;
    }

    location /server2/ {
        proxy_pass http://127.0.0.1:3000/;
    }

    location /sslopen {
        proxy_pass http://127.0.0.1:3000/;
    }
}
```

```bash
sudo systemctl restart nginx
```

---

### 🔒 Enable HTTPS

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

---

### 🛡️ Content Security Policy (CSP)

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self';";
```

---

## 🗄️ Database Security (MariaDB)

### Install & Secure
```bash
sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo mysql_secure_installation
```

---

### Create Database & User
```sql
CREATE DATABASE secure_onboarding;

CREATE USER 'secure_user'@'localhost' IDENTIFIED BY '123';

GRANT SELECT, INSERT, UPDATE, DELETE 
ON secure_onboarding.* 
TO 'secure_user'@'localhost';

FLUSH PRIVILEGES;
```

---

### Restrict Access
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```
bind-address = 127.0.0.1
```

---

### 📦 Database Backup

#### Script
```bash
sudo nano /usr/local/bin/db_backup.sh
```

```bash
#!/bin/bash
DATE=$(date +%F)
BACKUP_DIR="/var/backups/mariadb"

mysqldump -u root secure_onboarding > $BACKUP_DIR/db_$DATE.sql
gzip $BACKUP_DIR/db_$DATE.sql
```

#### Permissions
```bash
sudo chmod 700 /usr/local/bin/db_backup.sh
```

#### Schedule (3 AM)
```bash
sudo crontab -e
```

```
0 3 * * * /usr/local/bin/db_backup.sh
```

---

## 🔐 VPN Setup (WireGuard)

### Install
```bash
sudo apt install wireguard -y
```

### Generate Keys
```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

---

### Server Config
```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <server_private_key>

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.0.0.2/32
```

---

### Client Config
```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.0.0.2/24

[Peer]
PublicKey = <server_public_key>
Endpoint = <vm_ip>:51820
AllowedIPs = 0.0.0.0/0
```

---

### Start VPN
```bash
sudo wg-quick up wg0
sudo wg-quick up client1
```

### Allow VPN Port
```bash
sudo ufw allow 51820/udp
```

---

### Test Connection
```bash
ping 10.0.0.1
```

---

## 🎯 Summary

This setup provides:

- Secure SSH access (no passwords, no root login)  
- Firewall protection with strict rules  
- Intrusion detection using Suricata  
- Role-based access control  
- Automated backups (user + database)  
- Secure web hosting with HTTPS  
- Local-only database access  
- Private networking using WireGuard VPN  

---
