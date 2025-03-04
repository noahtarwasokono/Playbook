This is a guide start to finish
https://www.netwrix.com/linux_hardening_security_best_practices.html
https://www.cyberciti.biz/tips/linux-security.html
https://github.com/trimstray/linux-hardening-checklist/blob/master/README.md

1. Check `.bashrc` for malicious aliases, `source ~/.bashrc` to enable changes
2. Change your default password: `passwd <username>` will let you do this
3. Check users: look in `/etc/passwd` and `/etc/shadow` for users that shouldn't be there and can them. `userdel <username>`
4. Check groups: `/etc/group`
5. Check cronjobs: `/etc/crontab` is a file containing all running cron jobs. You should also check all the directories `/etc/cron.*` that are mentioned in the `crontab` file
6. Check SSH files: `cat ~/.ssh/authorized_keys` and just make sure there aren't any malicious keys in there. The black team will give us their authorized key
7. Check system binaries for files with suspicious names like `redteam`, `red_herring`, `dropbear`, `watershell`


Change default passwords
Remove bad users
Remove bad SSH keys
Give only certain users SSH access
Go through the conf files
Back up everything you can

`/etc/passwd`: Check if a user has `nologin` or `/bin/bash`
`/etc/shadow`: Check if a user has a hash
	If a user has `username:*:...`, they have no login
	If a user has `username::...`, they have no password
If you see a malicious user 

Pins: `/etc/apt/apt.conf` `/etc/apt/sources.list` `/etc/apt/preferences` `/etc/apt/preferences.d/`
Also you should turn on logging, which is usually `rsyslog` or `rsyslogd`

# Critical Actions for a compromised system in MikroTik (Assume they are already within the system)
```sh
# Reset compromised user credentials
/user set <username> password=<new-password> (MikroTik Router)

# Check for currently active users and terminate any unauthorized sessions
/user active print
/user active remove 1
/ip connection print
/ip firewall filter add chain=input src-address=<unauthorized-ip> action=drop
/log print
# Check scheduler for any automated tasks

# Review user groups and permissions; remove unauthorized users

```
