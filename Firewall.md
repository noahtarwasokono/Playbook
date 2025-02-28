This is specifically for firewalls in [[Linux]] lol, like [[ufw]], firewalld, and iptables. Since I already have a note for ufw, this is gonna focus on the harder ones

file:///C:/Users/njt12/Downloads/lecture8-CentOS7_RouterSetup%20(9).pdf

## iptables
iptables is less forgiving than ufw. If you turn it on when the default deny is on, it will instantly cut any ssh sessions. When you restart it, any unsaved rules are gone.

Commands
```sh
# check rules
sudo iptables -L
sudo apt install iptables-persistent
sudo iptables -P INPUT DROP # drops all input incoming (default deny in)
sudo iptables -P FORWARD DROP
# allow ping
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
# allow ssh in
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# -A is in or out, -p is protocol, --dport is dest port, -j ACCEPT DROP REJECT

# save rules
iptables-save > /etc/iptables/rules.v4
```
## firewalld 
Typically this is useful for machines that have zones, such as a router. For this reason, you use in in NCAE on the Router machine. Firewalld needs the permanent flag on rules you want to keep and must reload for them to take effect

https://www.karltarvas.com/centos-7-redirect-port-80-to-8080 

file:///C:/Users/njt12/Downloads/lecture8-CentOS7_RouterSetup%20(10).pdf - Set up as Router

Multi-Interface Configuration
```sh
# Find external and internal interfaces
vi /etc/sysconfig/network-scripts/ifcfg-eth0 
ZONE=external/internal
systemctl restart network

# IP Forwarding
sudo vi /etc/sysctl.d/ip_forward.conf
net.ipv4.ip_forward=1
sysctl -p /etc/sysctl.d/ip_forward.conf

# Enable IP Masquerading
sudo firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o
eth0 -j MASQUERADE -s ${INTERNAL_IP}/${NETMASK}
squerading
```

Let's go in order of the configuration we want for the router
```sh
# view all your zones
sudo firewall-cmd --list-all-zones | less # check interfaces and services in each zone
sudo firewall-cmd --list-all --zone=public # limit output to one zone

# change zones
sudo firewall-cmd --change-interface=eth1 --zone=external --permanent
sudo firewall-cmd --set-default-zone=internal
sudo firewall-cmd --zone=internal --set-target=DROP --permanent
sudo firewall-cmd --zone=external --set-target=DROP --permanent
sudo firewall-cmd --complete-reload

# port forwarding
sudo firewall-cmd --zone=external --add-forward-port=port=80:proto=tcp:toport=80:toaddr=192.168.118.2 --permanent
# forward TCP requests on port 80 to port 80 on 192.168.118.2

# remove external ssh connections to your firewall (test with your external machines)
sudo firewall-cmd --zone=external --permanent --remove-service=ssh
# add it back
sudo firewall-cmd --zone=external --permanent --add-service=ssh
# forwarding for the same service, will override potential to ssh to router
sudo firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=22:toaddr=192.168.118.2 --permanent
```
Reminder, external has masquerading enabled by default and this is necessary for an external zone

Setting DNS stuff for the router is cool, too
```sh
# change the port but make sure to change the protocol, too
sudo firewall-cmd --zone=external --add-forward-port=port=53:proto=udp:toport=53:toaddr=192.168.118.2 --permanent
```

Commands:
```sh
# check rules
sudo firewall-cmd --list-all-zones # without -zones is brief list
sudo firewall-cmd --list-forward-ports

# new default rule to DROP for all
sudo firewall-cmd --zone=public --set-target=DROP --permanent
# if you don't add permanent to the rule it will delete on reboot

# add ssh, it will recognize and map service to protocol
sudo firewall-cmd --zone=public --add-service=ssh --permanent

sudo firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.4:toport=80 --permanent

# restart firewalld
sudo firewall-cmd --reload
```

# Backup Firewall
```sh 
sudo apt install git -y  # Ubuntu/Debian  
sudo ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

# Go to GitHub > Settings > SSH and GPG keys. Click New SSH Key, paste the key, and save it.
cat ~/.ssh/id_rsa.pub
# Test SSH connection
ssh -T git@github.com

# Clone the Repository to Your Firewall Machine
mkdir -p ~/firewalld-backups
cd ~/firewalld-backups
git init
git remote add origin git@github.com:<your-username>/firewalld-backups.git

# Create Backup Script
sudo nano /root/firewalld_backup.sh
___________________________________________________________
#!/bin/bash

# Define variables
BACKUP_DIR="/root/firewalld_backups"
REPO_DIR="/root/firewalld-backups"
BACKUP_FILE="firewalld-$(date +%F_%H-%M).tar.gz"
ENCRYPTED_FILE="$BACKUP_FILE.gpg"
LOG_FILE="/var/log/firewalld_monitor.log"
GPG_RECIPIENT="your-email@example.com"  # Change this to your GPG key email
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/YOUR_WEBHOOK_HERE"  # Change this!

# Ensure backup directory exists
mkdir -p "$BACKUP_DIR"

# Backup firewalld configuration
tar -czf - /etc/firewalld | gpg --encrypt --recipient "$GPG_RECIPIENT" > "$BACKUP_DIR/$ENCRYPTED_FILE"

# Move encrypted backup to the Git repository
cp "$BACKUP_DIR/$ENCRYPTED_FILE" "$REPO_DIR"

# Check for unauthorized changes
cd "$REPO_DIR"
if ! git diff --quiet; then
    ALERT_MSG="🚨 WARNING: Firewall settings changed! Potential RED TEAM activity! 🚨"
    echo "$(date): $ALERT_MSG" | tee -a "$LOG_FILE"

    # Send Discord alert
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$ALERT_MSG\"}" "$DISCORD_WEBHOOK_URL"
fi

# Commit and push to GitHub with signed commit
git config user.name "Firewall Backup"
git config user.email "your-email@example.com"
git add "$ENCRYPTED_FILE"
git commit -S -m "Backup: $(date '+%Y-%m-%d %H:%M')"
git push origin main

# Keep only the last 10 backups in the repo
ls -t "$REPO_DIR" | grep "firewalld-.*.tar.gz.gpg" | tail -n +11 | xargs -I {} rm -- "$REPO_DIR/{}" 2>/dev/null || true
___________________________________________________________
sudo chmod +x /root/firewalld_backup.sh
sudo crontab -e
*/2 * * * * /root/firewalld_backup.sh >> /var/log/firewalld_backup.log 2>&1
*/2 * * * * git -C /root/firewalld-backups diff --quiet || echo "$(date): FIREWALL CHANGED!" >> /var/log/firewalld_monitor.log
tail -f /var/log/firewalld_backup.log

# Test Alert to Discord
# Open Discord and go to the server where you want to receive alerts.
# Click Server Settings > Integrations > Create Webhook.
# Name it (e.g., Firewall Alerts), choose a channel, and copy the Webhook URL.
echo "test" >> /etc/firewalld/testfile

gpg --full-generate-key
git config --global user.signingkey <your-key-id>
git config --global commit.gpgsign true

# Immutable Logs
sudo chattr +i /var/log/firewalld_monitor.log
# Restore Encrypted Backup
gpg --decrypt /root/firewalld_backups/firewalld-<DATE>.tar.gz.gpg | tar -xz
```
