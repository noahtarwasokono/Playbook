/system identity set name=SecureRouter

# Change password for the current user
/user set 0 password="YourNewSecurePassword"

‎```sh
# Disable unnecessary services
ip service disable telnetftp,etc
ip service set ssh port=2200
ip ssh set strong-crypto=yes
ip service set winbox address=x.x.x.x/24
/ip proxy set enabled=no
‎```sh

# Set static IPs
/ip address print 
/ip route print
/interface print
/ip address add address=192.168.1.1/24 interface=ether1 comment="LAN"
/ip address add address=172.18.0.1/16 interface=ether2 comment="WAN"
/ping x.x.x.x

# Set up SSL certificate (assuming you have an existing cert)
/certificate
add name=Server common-name=server
sign Server
print
print detail
.. # exit
/ip service print detail
/ip service enable www-ssl
/ip service set www-ssl certificate=Server disabled=no
/ip service set www-ssl address=192.168.161.2

/ip ssl set certificate=mySSL.crt key=mySSL.key
/ip service set www-ssl certificate=mySSL.crt disabled=no

# User audit: Create a new user and disable admin/root
/user print
/user set <idx or username> password="password"
/user set <idx or username> address=x.x.x.x/yy
/user add name=secureuser password="NewUserPass" group=full
/user set admin disabled=yes or /user disable admin

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

