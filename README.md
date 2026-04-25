# 🔐 Secure Cloud VM Setup & Hardening (Azure Ubuntu 22.04)

This project demonstrates the complete setup and security hardening of an Ubuntu 22.04 Virtual Machine on Azure, including firewall configuration, intrusion detection, secure user management, backups, web server deployment, database security, VPN setup, Docker containerization, and Ansible automation.

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

## 🐳 Stage 8: Docker Containerization

### Part 1: Basic Docker Setup

#### 1️⃣ Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
```
> Installs Docker engine — allows you to run containers.

#### 2️⃣ Start and Enable Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```
> `start` → runs Docker now. `enable` → auto-starts Docker on every boot.

#### 3️⃣ Add User to Docker Group
```bash
sudo usermod -aG docker $USER
newgrp docker
```
> Without this, every Docker command requires `sudo`. After adding to the group, you can run `docker run ...` directly without elevated privileges.

#### 4️⃣ Test Docker
```bash
docker run hello-world
```
> Expected output: `Hello from Docker!` — confirms Docker is working correctly. ✅

---

### Part 2: Create Portfolio Website

#### 1️⃣ Create Project Folder
```bash
mkdir ~/portfolio
cd ~/portfolio
```

#### 2️⃣ Create the HTML Site
```bash
nano index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Portfolio</title>
</head>
<body>
    <h1>Welcome to My Portfolio</h1>
    <p>This is running inside Docker 🚀</p>
</body>
</html>
```
> This is the website content that will be served from inside the container.

---

### Part 3: Containerize the Website

#### 1️⃣ Create Dockerfile
```bash
nano Dockerfile
```

```dockerfile
FROM nginx:latest
COPY . /usr/share/nginx/html
```

| Line | Meaning |
|------|---------|
| `FROM nginx` | Use the official Nginx image as base |
| `COPY` | Copy your site files into the container's web root |

#### 2️⃣ Build the Image
```bash
docker build -t portfolio-site .
```
> Creates a custom Docker image named `portfolio-site` from the Dockerfile.

#### 3️⃣ Run the Container
```bash
docker run -d -p 8080:80 --name portfolio portfolio-site
```

| Flag | Meaning |
|------|---------|
| `-d` | Run container in background (detached mode) |
| `-p 8080:80` | Map host port 8080 to container port 80 |
| `--name` | Assign a name to the container |

#### 🧪 Test
```bash
curl http://localhost:8080
```
> Should return your HTML content.

---

### Part 4: Use a Volume

Stop and remove the existing container first:
```bash
docker stop portfolio
docker rm portfolio
```

Run with a bind mount volume:
```bash
docker run -d \
  -p 8080:80 \
  --name portfolio \
  -v ~/portfolio:/usr/share/nginx/html \
  nginx
```
> The volume maps your local `~/portfolio` folder directly into the container. Any edits to the HTML are instantly reflected without rebuilding the image. Data also persists even if the container is stopped.

---

### Part 5: Auto-start Container on Boot

```bash
docker run -d \
  -p 8080:80 \
  --name portfolio \
  --restart always \
  -v ~/portfolio:/usr/share/nginx/html \
  nginx
```
> `--restart always` ensures the container automatically restarts whenever the VM reboots.

---

### Part 6: Reverse Proxy via Host Nginx

Edit the Nginx config to forward traffic to the Docker container:
```bash
sudo nano /etc/nginx/sites-available/default
```

Add inside the `server` block:
```nginx
location /portfolio/ {
    proxy_pass http://127.0.0.1:8080/;
}
```
> This routes requests to `/portfolio/` on the public IP into the Docker container running on port 8080.

Restart Nginx to apply changes:
```bash
sudo systemctl restart nginx
```

#### 🧪 Test
Open in browser:
```
http://YOUR_IP/portfolio/
```
> Your portfolio website should appear.

---

### Part 7: Fix Permissions

```bash
chmod -R 755 ~/portfolio
```
> Ensures the Nginx process inside the container can read all files in the portfolio directory.

---

## 🤖 Stage 9: Ansible Automation

### Part 1: Setup Ansible Lab

#### 🔹 Step 1: Create Docker Network
```bash
docker network create ansible-net
```
> Creates an isolated network so containers can communicate with each other by name (e.g., `target1`, `target2`) instead of IP addresses.

#### 🔹 Step 2: Create Target Containers (Simulated SSH Servers)

Run target1:
```bash
docker run -dit --name target1 --network ansible-net ubuntu:22.04
```

Run target2:
```bash
docker run -dit --name target2 --network ansible-net ubuntu:22.04
```
> These containers simulate remote servers that Ansible will manage.

#### 🔹 Step 3: Install SSH in Target Containers

Enter the container:
```bash
docker exec -it target1 bash
```

Inside the container, run:
```bash
apt update
apt install openssh-server -y
mkdir /var/run/sshd
echo 'root:root' | chpasswd
```

Edit SSH config to allow root and password login:
```bash
nano /etc/ssh/sshd_config
```

```
PermitRootLogin yes
PasswordAuthentication yes
```

Start the SSH daemon:
```bash
/usr/sbin/sshd
```

> Repeat the same steps for `target2`. SSH is required so Ansible can connect to and manage these containers.

#### 🔹 Step 4: Create Control Container
```bash
docker run -dit --name control --network ansible-net ubuntu:22.04
docker exec -it control bash
```

#### 🔹 Step 5: Install Ansible in Control Container
```bash
apt update
apt install ansible openssh-client -y
```
> `ansible` provides the automation engine. `openssh-client` enables SSH connections to the target containers.

#### 🔐 Step 6: Setup SSH Key Authentication

Inside the control container, generate a key pair:
```bash
ssh-keygen
```
Press Enter for all prompts.

Copy the public key to each target:
```bash
ssh-copy-id root@target1
ssh-copy-id root@target2
```
(Password: `root`)

> Key-based auth avoids password prompts during automated playbook runs.

#### 🔹 Step 7: Create Inventory File
```bash
nano inventory.ini
```

```ini
[targets]
target1
target2
```
> The inventory file tells Ansible which machines to manage.

#### 🔹 Step 8: Test Ansible Connection
```bash
ansible -i inventory.ini targets -m ping
```

Expected output:
```
target1 | SUCCESS
target2 | SUCCESS
```

#### 🔹 Step 9: Basic Playbook
```bash
nano test.yml
```

```yaml
- hosts: targets
  tasks:
    - name: Ping
      ping:

    - name: Disk usage
      command: df -h

    - name: Uptime
      command: uptime
```

Run the playbook:
```bash
ansible-playbook -i inventory.ini test.yml
```
> This confirms that Ansible can connect to both targets and execute tasks automatically.

---

### Part 2: Configuration Management

#### 🔹 Step 1: Create Advanced Playbook
```bash
nano setup.yml
```

```yaml
- hosts: targets
  become: yes

  vars:
    packages:
      - python3
      - git
      - vim
      - htop
      - curl
      - wget
      - tmux

  tasks:

    - name: Install packages
      apt:
        name: "{{ packages }}"
        state: present
        update_cache: yes

    - name: Set bash alias
      copy:
        dest: /etc/profile.d/custom.sh
        content: |
          alias ll='ls -la'

    - name: Configure vim
      copy:
        dest: /etc/vim/vimrc.local
        content: |
          set number

    - name: Set permissions
      file:
        path: /home
        mode: '755'
```

| Feature | Purpose |
|---------|---------|
| `vars` | Reusable values (package list) |
| `tasks` | Actual work to be performed |
| `copy` | Push configuration files to targets |

#### 🔐 Add UFW Firewall & SSH Hardening

Append to `setup.yml`:
```yaml
    - name: Install UFW
      apt:
        name: ufw
        state: present

    - name: Allow SSH
      ufw:
        rule: allow
        port: 22

    - name: Enable firewall
      ufw:
        state: enabled
```
> Automates security configuration across all target machines in one run.

---

### Part 3: Ansible Roles

#### 🔹 Step 1: Generate Role Scaffolding
```bash
ansible-galaxy init lab-base
ansible-galaxy init student-workstation
```

#### 🔹 Role 1: lab-base

Edit the tasks file:
```bash
nano lab-base/tasks/main.yml
```

```yaml
- name: Update packages
  apt:
    update_cache: yes

- name: Install monitoring tool
  apt:
    name: htop
    state: present
```
> Sets up the base environment on all lab machines.

#### 🔹 Role 2: student-workstation

Edit the tasks file:
```bash
nano student-workstation/tasks/main.yml
```

```yaml
- name: Create student user
  user:
    name: student
    state: present

- name: Install dev tools
  apt:
    name:
      - git
      - python3
    state: present
```
> Simulates a developer/student lab environment with required tools and a user account.

#### 🔹 Step 3: Apply Roles via Playbook
```bash
nano roles.yml
```

```yaml
- hosts: targets
  become: yes
  roles:
    - lab-base
    - student-workstation
```

Run:
```bash
ansible-playbook -i inventory.ini roles.yml
```

#### 🧠 Idempotence

Running the playbook multiple times is safe:
```bash
ansible-playbook -i inventory.ini roles.yml
```
> Ansible only makes changes when the current state differs from the desired state. Running the same playbook twice produces no unintended side effects — this is called **idempotence**.

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
- Containerized web application using Docker  
- Infrastructure automation using Ansible with roles and playbooks  

---
