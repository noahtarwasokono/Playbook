/system identity set name=SecureRouter

# Change password for the current user
/user set 0 password="YourNewSecurePassword"

# Disable unnecessary services
/ip service set telnet disabled=yes
/ip service set ftp disabled=yes
/ip service set www disabled=yes
/ip service set www-ssl disabled=yes
/ip service set api disabled=yes
/ip service set api-ssl disabled=yes

# Set static IPs
/ip address add address=192.168.1.1/24 interface=ether1 comment="LAN"
/ip address add address=172.18.0.1/16 interface=ether2 comment="WAN"

# Set up SSL certificate (assuming you have an existing cert)
/ip ssl set certificate=mySSL.crt key=mySSL.key
/ip service set www-ssl certificate=mySSL.crt disabled=no

# User audit: Create a new user and disable admin/root
/user add name=secureuser group=full password="NewUserPass"
/user set admin disabled=yes

# Restrict SSL access to a specific IP
/ip firewall filter add chain=input protocol=tcp dst-port=443 src-address=192.168.1.100 action=accept comment="Allow SSL from specific IP"
/ip firewall filter add chain=input protocol=tcp dst-port=443 action=drop comment="Block all other SSL access"

# Harden the machine further (adjust as needed)
/system logging set action=memory
/system logging add topics=info action=memory

/system script add name="HardenRouter" source="\
    /user set admin disabled=yes; \
    /ip firewall filter add chain=input connection-state=invalid action=drop comment=\"Drop invalid connections\"; \
    /ip firewall filter add chain=input protocol=icmp action=accept comment=\"Allow ICMP\"; \
    /ip firewall filter add chain=input in-interface=ether1 action=accept comment=\"Allow LAN traffic\"; \
    /ip firewall filter add chain=input action=drop comment=\"Drop everything else\"; \
"
/system script run HardenRouter

