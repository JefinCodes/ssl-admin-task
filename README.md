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













