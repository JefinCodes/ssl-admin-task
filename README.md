##Initial Setup
Setup azure student account
created virtual machine of ubuntu 22.04 image
attach it with dns name jefincodes.koreacentral.cloudapp.azure.com
refresh list of available packages from repositories - sudo apt update
install latest version of all installed packages - sudo apt upgrade

What “unattended upgrades” does
Unattended Upgrades is a tool that:
Automatically checks for security updates
Downloads them
Installs them automatically
Runs without human intervention

unattended-upgrade was already setup

actually unattended-upgrade is a tool itself. Its scheduled once a day and it checks for pending security updates and if any pending found then downloads and installs it automaticaly

##Enhanced SSH Security
A user was already there with sudo privilleges ie, was member of sudo group
Setup SSH connection using public private key
Disable root login
By default root login was allowed using key
Changed settings to disallow root login using password as well as key
Disable password authentication for all users


##Firewall and Network Security

reset all rules to default
sudo ufw reset

Allow all outgoing and block all incoming traffic
sudo ufw default deny incoming
sudo ufw default allow outgoing

Allow ssh, http, https ports
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

Enable ufw
sudo ufw enable

Check Status
sudo ufw status verbose

Enable Logging
sudo ufw logging on

more detailed logging
sudo ufw logging medium

to view logs
sudo tail -f /var/log/ufw.log

Setting up Intrusion Detection System

Update system and install suricata
sudo apt update
sudo apt install suricata -y

start and enable
sudo systemctl enable suricata
sudo systemctl start suricata

check status
sudo systemctl status suricata

suricata logs will be stored in
/var/log/suricata/

Check alerts
sudo tail -f /var/log/suricata/fast.log

Install rules
sudo suricata-update

Restart
sudo systemctl restart suricata

##User and Permission Management

Create exam users
sudo adduser exam_1
sudo adduser exam_2
sudo adduser exam_3

create admin user
sudo adduser examadmin

Add examadmin user to sudo group
sudo usermod -aG sudo examadmin

Create audit user
sudo adduser examaudit

Create a audit group
sudo groupadd auditgroup

Add examaudit user to auditgroup
sudo usermod -aG auditgroup examaudit

Add home directories of all exam users to auditgroup
sudo chgrp auditgroup /home/exam_1
sudo chgrp auditgroup /home/exam_2
sudo chgrp auditgroup /home/exam_3

what we are doing is we are adding home directories of all exam users to audit group where examaudit user is present. But we are no adding exam users to this group. So finally we set group permissions of home directories of exam users to read + execute. So examaudit user can read all exam users home directories

sudo chmod 750 /home/exam_1
sudo chmod 750 /home/exam_2
sudo chmod 750 /home/exam_3

Backup Script

Create script file
sudo nano /usr/local/bin/exam_backup.sh

backup script
#!/bin/bash
BACKUP_DIR="/var/backups/exam_users"
DATE=$(date +%F)
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/exam_backup_$DATE.tar.gz /home/exam_*

What the script does? creates backup directory if already not created. gets all home directories starting with exam_ and places it into a folder and compress that folder and save it with name exam_backup_$DATE.tar.gz

set backup script file permissions
sudo chown examadmin:examadmin /usr/local/bin/exam_backup.sh
sudo chmod 700 /usr/local/bin/exam_backup.sh

set backup directory permissions
sudo mkdir -p /var/backups/exam_users
sudo chown examadmin:examadmin /var/backups/exam_users
sudo chmod 700 /var/backups/exam_users

Schedule daily backup
Edit cron:
crontab -e
Add cron task to run exam backup script at 2 AM daily
0 2 * * * /usr/local/bin/exam_backup.sh

##Web Server Deployment and Secure Configuration

Create Non-Privileged User
sudo adduser appuser

Install Nginx
sudo apt update
sudo apt install nginx -y

Start
sudo systemctl enable nginx
sudo systemctl start nginx

Switch user
sudo su - appuser

Download app and verify signature
wget https://do.edvinbasil.com/ssl/app
wget https://do.edvinbasil.com/ssl/app.sha256.sig

Run app 1
chmod +x app
./app

Run app 2
git clone https://gitlab.com/tellmeY/issslopen.git
cd issslopen
bun install
bun run index.ts

This apps run on localhost and is only accessible inside vm

Configure Nginx Reverse Proxy
Edit config:
sudo nano /etc/nginx/sites-available/default
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

Restart Nginx
sudo systemctl restart nginx

Apps were publicly accessible
app 2 is made only locally accessible by editing the index.ts
but app 1 is found to be globally accessible when chacked using sudo ss -tulnp, but since firewall blocks every port except 22 80 443 there is no issue

Enable HTTPS
Install
sudo apt install certbot python3-certbot-nginx -y
Run
sudo certbot --nginx

Content Security Policy (CSP)
Add CSP in Nginx
Edit config:
sudo nano /etc/nginx/sites-available/default
Add inside server {}:
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self';";

##Database Security

Database Setup
sudo apt update
sudo apt install mariadb-server -y

Start it:
sudo systemctl enable mariadb
sudo systemctl start mariadb

Secure initial installation
sudo mysql_secure_installation

In the intial setup itself we can disable root login

Login to MariaDB
sudo mysql

Create database
CREATE DATABASE secure_onboarding;

Create minimal-privilege user
CREATE USER 'secure_user'@'localhost' IDENTIFIED BY '123';

Grant limited permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON secure_onboarding.* TO 'secure_user'@'localhost';

Apply changes
FLUSH PRIVILEGES;
EXIT;

Restrict MariaDB to localhost ONLY
Check current binding:
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
Find:
bind-address = 127.0.0.1
So local access only

automated backups

backup directory
sudo mkdir -p /var/backups/mariadb
sudo chown jefincodes:jefincodes /var/backups/mariadb
sudo chmod 700 /var/backups/mariadb

Create backup script
sudo nano /usr/local/bin/db_backup.sh

Script:
#!/bin/bash
DATE=$(date +%F)
BACKUP_DIR="/var/backups/mariadb"
mysqldump -u root secure_onboarding > $BACKUP_DIR/db_$DATE.sql
gzip $BACKUP_DIR/db_$DATE.sql

Secure script
sudo chmod 700 /usr/local/bin/db_backup.sh

Schedule with cron
sudo crontab -e

Add:
0 3 * * * /usr/local/bin/db_backup.sh

Runs daily at 3 AM

##VPN Configuration

Install WireGuard on client and server
sudo apt install wireguard -y

Create identity (keys) on client and server
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key

Create VPN network
in server
sudo nano /etc/wireguard/wg0.conf
in client
sudo nano /etc/wireguard/client1.conf

Add:
In server
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = server private key
[Peer]
PublicKey = client public key
AllowedIPs = 10.0.0.2/32
In client
[Interface]
PrivateKey = client private key
Address = 10.0.0.2/24
[Peer]
PublicKey = server public key
Endpoint = vm_ip:51820
AllowedIPs = 0.0.0.0/0

Start VPN
in server
sudo wg-quick up wg0
in client
sudo wg-quick up client1

allow vpn port 51820 in vm firewall as well in azure settings
sudo ufw allow 51820/udp

Test if vpn connection successful by pinging from client to server
ping 10.0.0.1
