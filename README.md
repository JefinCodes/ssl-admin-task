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
















