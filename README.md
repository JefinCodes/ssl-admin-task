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
