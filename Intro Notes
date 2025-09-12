High Priority Services:
Service ID 4: SSH Service (Port 22, Linux)
Reason for High Priority: SSH is a crucial entry point for system management. If compromised, attackers can gain full control over the machine.
How to Fix: Change the default credentials immediately from the weak vagrant:vagrant to a strong password.
Mitigate: Restrict SSH access by limiting it to specific IP addresses (e.g., via firewall rules).
Command (Linux Firewall Example):
bash
Copy code
sudo ufw allow from [trusted_ip] to any port 22
sudo ufw enable
Harden: Disable root login and use key-based authentication:
Edit /etc/ssh/sshd_config:
bash
Copy code
PermitRootLogin no
PasswordAuthentication no

Restart SSH:
bash
Copy code
sudo systemctl restart ssh

Service ID 10: Remote Desktop (Port 3389, Windows)
Reason for High Priority: RDP is a common attack vector for Windows systems and is critical if the attacker gains access.
How to Fix: Change the weak credentials (greedo:hanSh0tF1rst) to a strong password. Disable RDP if not required for the competition.
Mitigate: Restrict access to RDP by allowing only trusted IPs via Windows Firewall.
Command:
powershell
Copy code
New-NetFirewallRule -DisplayName "Allow RDP from trusted IP" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3389 -RemoteAddress [trusted_ip]


Harden: Enable Network Level Authentication (NLA) for RDP and consider using two-factor authentication if possible.
Service ID 11: File Sharing (Port 445, SMB, Windows)
Reason for High Priority: SMB shares are commonly exploited in attacks like WannaCry. Default credentials are vagrant:vagrant, which are highly insecure.
How to Fix: Change credentials immediately, restrict access to only trusted IPs, and use the least privilege model for shared directories.
Mitigate: Block public access to SMB from external networks using firewall rules.
Harden: Disable SMBv1 to prevent exploitation of old vulnerabilities like EternalBlue.
Command:
powershell
Copy code
Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol

Service ID 3: FTP Service (Port 21, Linux)
Reason for High Priority: FTP transfers files in plain text, making it susceptible to sniffing and man-in-the-middle attacks. Default credentials (vagrant:vagrant) pose a significant risk.
How to Fix: Change default credentials immediately. Consider switching to a secure alternative like SFTP.
Mitigate: Restrict access to the FTP server, limit connections to trusted IPs, and disable anonymous FTP access.
Harden: Use encryption (FTP over TLS) or replace with a more secure protocol like SFTP.
Service ID 7: FTP Service 2 (Port 21, Windows)
Reason for High Priority: Like the Linux FTP service, the Windows FTP service has default credentials (vagrant:vagrant), making it highly vulnerable.
How to Fix: Change default credentials and consider disabling the service if it's not essential.
Mitigate: Restrict access to trusted IPs via firewall and implement stronger authentication methods.
Harden: Move to a more secure file transfer protocol like SFTP.

Medium Priority Services:
Service ID 1: Drupal Website (Port 80, Linux)
Reason for Medium Priority: While web applications are common targets for attacks like SQL injection, Drupal’s default credentials (metasploitable:newpwd) present a significant risk.
How to Fix: Change credentials immediately and ensure all plugins and themes are updated.
Mitigate: Disable any unnecessary modules in Drupal, and restrict access to the admin page.
Harden: Enable a Web Application Firewall (WAF) and make sure Drupal core is up to date.
Service ID 9: Elastic Search (Port 9200, Windows)
Reason for Medium Priority: ElasticSearch, by default, is accessible over HTTP without authentication, making it susceptible to data exfiltration or unauthorized access.
How to Fix: Add authentication and encryption (HTTPS) to the ElasticSearch instance. Change default credentials immediately.
Mitigate: Restrict access to ElasticSearch via firewall, limiting to specific trusted IP addresses.
Harden: Regularly patch ElasticSearch, enable access control (role-based access), and use a reverse proxy to limit exposure.
Service ID 6: Info Website (Port 3500, Linux)
Reason for Medium Priority: This service could contain sensitive information that may be abused if left exposed. While not critical, it's important to secure.
How to Fix: Update website files, change any default credentials, and patch any vulnerabilities in the codebase.
Mitigate: Use a firewall to restrict public access to the web service.
Harden: Use SSL/TLS (HTTPS) to encrypt traffic.

Low Priority Services:
Service ID 12: Blog (Port 8585, Windows)
Reason for Low Priority: A blog on a non-standard port might be less likely to be directly targeted, but weak credentials (vagrant:vagrant) make it vulnerable.
How to Fix: Change the default credentials immediately and review the security of the blog CMS (likely WordPress or similar).
Mitigate: Restrict access via firewall and keep the CMS up to date with security patches.
Harden: Implement HTTPS, install a security plugin, and enforce strong password policies.
Service ID 8: Turtle Website (Port 80, Windows)
Reason for Low Priority: The service has been defaced, and while it needs to be fixed, it's less of a priority compared to other services that are critical for system access or sensitive data.
How to Fix: Restore the website to its original state and change any exposed credentials or configurations.
Mitigate: Ensure web application security by patching vulnerabilities and reviewing website code for weaknesses.
Harden: Implement a Web Application Firewall (WAF) and force HTTPS to secure traffic.
Service ID 5: IRC Service (Port 6697, Linux)
Reason for Low Priority: While IRC can be used for communication, it's less critical compared to other services like FTP, SSH, and web applications.
How to Fix: Ensure that the service has strong authentication mechanisms and restrict public access to the IRC server.
Mitigate: Only allow trusted IPs to connect to the IRC service.
Harden: Implement SSL for secure communication and monitor for unusual traffic.

General Security Practices to Apply Across All Services:
Change Default Credentials: Every service has default credentials that must be changed immediately to something secure.
Enable Firewalls: Ensure that the firewall is configured on both Linux and Windows to restrict access to services only from trusted IPs.
Use Strong Authentication: Implement strong password policies and, where applicable, use two-factor authentication (2FA).
Apply Patches and Updates: Ensure that all services and the underlying operating systems are fully patched with the latest security updates.
Monitor Logs: Continuously monitor service logs (e.g., /var/log/apache2/access.log or Windows Event Logs) for signs of compromise or unusual behavior.
Backup Critical Services: Ensure backups are in place for critical services (e.g., web applications, databases) so that they can be restored in the event of an attack.

Hardening

Injects
Fire walls, W
Prevent ssh 22
Disable users, don’t delete
Keeping logs
Get backups going
Find indicators of compromise
The hardening never finishes
Get creative, create configs for intrusion detection systems, 
Study penetrating tutorials
SSH and … general services, file transfers, mail servers?
Setup open ssh server
Change defaults credentials change malicious configs, tinker with the vms 
Setup firewalls
Audit users and groups
Back up critical services
There are many things you can do to harden
You need to know what users are on the system and what they can do
Know which services need to be up
Dependency for a scordes service lock it down
Change default credentials
Changeme123 is default
File and folder access permissions, lock down folders 
Learn how to open and close ports, look at what services are running and what we can disable
Enumerate your services figure out what’s going on and lock it down
Lock down rdp
What apps can users run, disable bad ones
Network shares
When a usc prop shows up, when a user elevates privileges
MAINTAIN BACKUPS
RDP is a score service, how to allow it?
disable RDP unless it is needed. Blocking port 3389 using a firewall can also help.
TURN THE FIREWALL ON INSTANTLY
We can’t fix everything
