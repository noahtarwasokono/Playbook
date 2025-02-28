Keeping bad guys out is as important as getting your infrastructure up. Servers don't help
when they get knocked offline. To help you track down those pesky Red Teamers, use the
information below as reference for some areas you may find anomolies.
(HINT: Often, IT and SecOPs teams work together to keep services functioning and
secure. If your infrastructure is in good shape, a dedicated 'analyst' might help you in the
long run)

TO-DO
1. netstat, find reverse shells, other users
2. Running processes
3. Reboot
4. Check logs

# Protect
```sh
#Check services
service --status-all
#Service information
ps -aux
#Check for start jobs
ls /etc/init/*.conf
#Backup existing rules
iptables-save > iptables_rules.out
#Modify export if needed
vi iptables_rules.out
#Restore export
iptables-restore < firewall.out
#List rules
iptables -L
#Change password
passwd
```
# Detect
```sh
#*NETWORK*
#Look at live network traffic
tcpdump
#Save PCAP to remote host (Kali?)
tcpdump -w - | ssh <remote ip address here> -p <port> "cat - >
/tmp/<filename>.pcap"
#Monitor for new tcp connections (netstat has to be installed, yum install
net-tools)
netstat -ac 5 | grep tcp
#Monitor traffic remotely (from kali?)
ssh <user>@<remote ip of host to monitor> tcpdump -i any -U -s -) -w - 'not
host <kali ip>'
#*LOGS* (look below for common linux logs)
#Look at log
tail /path/to/log
#Look at log in real time
tail -f /path/to/log
#Look for keyword in log
grep -i "<keyword>" /path/to/log
#Look for sudo activity
grep -i sudo /var/log/auth.log
```

# Triage
```sh
#View logged in users
w
#Check remote login activity
lastlog
#Check failed logins
faillog -a
#View local accounts and groups
cat /etc/passwd
cat /etc/shadow
cat /etc/group
cat /etc/sudoers
#Show root accounts
awk -F: '($3 == "0") {print}' /etc/passwd
#Active network connections
netstat -antup
#View routes
route
#List processes listening on ports
lsof -i
#Check cron
crontab -l
cat /etc/crontab
ls /etc/cron.*
#Stop a process (hint, use a command above for checking process info)
kill <process pid>
kill -9 -I <process name>
#Remove execution from a process (or just delete it)
chmod -x /path/to/malicious/file
#Move the malicious file to analyze it for potential attacker information
mv /path/to/malicious/file ~/quarantine
strings ~/quarantine/<malicious_file>
```
# Other commands
```sh
`ps aux`
`lsof -i`
`who`
`netstat -tulna`

`loginctl`
`loginctl terminate-session <sid>`

Firewall pins
`/etc/yum/yum.conf` or `/etc/yum.conf` will have a line that is formatted like the following with all package pins:
`exclude=apache2 firewalld iptables ufw`
When you remove a pin, you have to usually run `yum update` to have that take effect
The exclude statement might be in any of the following locations:
- `/etc/yum/yum.conf` or `/etc/yum.conf`
- `/etc/yum.repos.d/*`
- `/etc/yum/protected.d/*`
- `/etc/yum/yum.conf.d/*`
You may need to `yum update` to get it working from there
If you're working with DNF, try checking here:
- `/etc/dnf/protected.d/*`
- `/etc/dnf/dnf.conf.d/*`
These files will have a line that looks like `excludepkgs=*`
You can also use versionlock by running something like:
```sh
sudo dnf install dnf-plugins-core
sudo dnf versionlock list
sudo dnf versionlock delete <package-name>
```

Services:
You can find daemon files in `/lib/systemd/system`

Keyloggers:
Typically password loggers will be a line in the `/etc/pam.d/` folder and often in the `common-auth` file specifically
Or in Rocky or CentOS it'll be in `password-auth`

You can manually exit a specific aspect of a function by opening `gdb -p $(pid)` and running `call close($(file descriptor)` and then `exit`. You can also probably pull `gdb` without `apt` or `yum` by just `curl`ing it

Is psid 1 actually `init`? Is `init` actually pointing to `/lib/systemd/systemd`? Fun questions to ask
