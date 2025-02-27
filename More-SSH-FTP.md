# **Install SSH on the server by installing the openssh-server package** (INSTALLATION AND CONFIGURATION)
```sh
sudo apt update (Update Package Lists)
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh (verify ssh is running)
**If needed to allow SSH in firewall**
(sudo firewall-cmd --permanent --add-service=ssh)
(sudo firewall-cmd --reload) 
```

# **Generate an SSH key on machine, copy public key over to our server to blueteam account, connect to
server using key instead of password** (KEY GENERATION AND MANAGEMENT)
```sh
ssh-keygen -t rsa -b 4096 -C "lab-3-kali" (saved to somewhere in /home/username/.ssh/id_rsa)
ssh-copy-id blueteam@<server_ip>
ssh blueteam@<server_ip> (log into server)
```

# **SSH Hardening**
```sh
sudo nano /etc/ssh/ssh_config (open the ssh configuration file)
MaxAuthTries 3 (add or uncomment this)
ClientAliveInterval 60
ClientAliveCountMax 0
MaxSessions 5
```

# **SSH File Transfers**
```sh
scp ~/Documents/ssh-file.txt blueteam@server_ip:/home/blueteam/P4/
ssh into blueteam and cd /home/blueteam/P4/ and ls to confirm

**Logging and Monitoring SSH Activity**
SSH LOGS LOCATED IN /var/log/auth.log, use grep for failed/successful login attempts

grep 'Failed password' /var/log/auth.log | grep 'bob' | wc -l
echo "bob-failed:<count>" >> /home/blueteam/P5/P5.txt

grep 'Accepted password' /var/log/secure | grep 'redteam' | wc -l
echo "redteam-successful:<count>" >> /home/blueteam/P5/P5.txt

grep 'Accepted password' /var/log/secure | grep 'bob' | wc -l
echo "bob-password:<count>" >> /home/blueteam/P5/P5.txt

grep 'Accepted publickey' /var/log/secure | grep 'redteam' | wc -l
echo "redteam-key:<count>" >> /home/blueteam/P5/P5.txt
```

# **Troubleshooting**
```sh
sudo ssh -t (verify syntax of ssh config file)
sudo nano /etc/ssh/sshd_config
(check for syntax errors, etc)
sudo systemctl restart ssh
sudo systemctl status ssh (check status)
sudo journalctl -u ssh (check logs if needed)
```

# **FTP Installation, Configuration, and User Management**
```sh
sudo apt update -y
sudo apt install vsftpd -y
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo systemctl status vsftpd
sudo nano /etc/vsftpd.conf
chroot_local_user=YES
local_enable=YES
write_enable=YES
sudo systemctl restart vsftpd
sudo useradd -m ftpuser
sudo passwd ftpuser
echo ~ftpuser (make sure in home directory)
(FOR FIREWALL IN FUTURE - sudo ufw allow ftp, sudo ufw status)
ftp <server_ip>
```

# *FTP Security* - Disable anonymous login to the FTP server
```sh
sudo nano /etc/vsftpd.conf
anonymous_enable=NO
sudo systemctl restart vsftpd
```

