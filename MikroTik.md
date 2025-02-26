# Ensure clean router before configuring anything else
/system reset-configuration no-defaults=yes skip-backup=yes


```sh
/system identity set name=SecureRouter

# Change password for the current user
/user set 0 password="YourNewSecurePassword"
# Use password generator tool, below is more secure
/password

/user remove admin
```

‎
# Disable unnecessary services
```sh
ip service disable telnetftp,etc
# Stop random SSH brute force login attempts
ip service set ssh port=2200
ip ssh set strong-crypto=yes
# Limit service to certain IPs
ip service set winbox address=x.x.x.x/24
/ip proxy set enabled=no

# Block all Unwanted Access
/ip firewall filter add chain=input protocol=tcp dst-port=8291 action=drop comment="Block Winbox from WAN"
/ip firewall filter add chain=input protocol=tcp dst-port=22 action=drop comment="Block SSH from WAN"
/ip firewall filter add chain=input protocol=tcp dst-port=80 action=drop comment="Block HTTP from WAN"
/ip firewall filter add chain=input protocol=tcp dst-port=443 action=drop comment="Block HTTPS from WAN"
/ip firewall filter add chain=input protocol=tcp dst-port=8728 action=drop comment="Block API from WAN"
/ip firewall filter add chain=input protocol=tcp dst-port=8729 action=drop comment="Block API-SSL from WAN"

# Other services
/tool bandwidth-server set enabled=no
/ip dns set allow-remote-requests=no
/lcd set enabled=no
/interface print 
/interface set x disabled=yes # Disable any unused interfaces
/ip proxy set enabled=no # MikroTik caching proxy
/ip socks set enabled=no # MikroTik socks proxy
/ip upnp set enabled=no # MikroTik UPNP service
/ip cloud set ddns-enabled=no update-time=no # MikroTik dynamic name service or IP cloud
```

# NAT Configuration (change source address to router public IP)
```sh
/ip firewall nat
  add chain=srcnat out-interface=ether1 action=masquerade
```

# Port Forwarding (if client device needs access over certain port)
```sh
/ip firewall nat
  add chain=dstnat protocol=tcp port=3389 in-interface=ether1 \
    action=dst-nat to-address=192.168.88.254
```

```sh
# Set static IPs
/ip address print 
/ip route print
/interface print
/ip address add address=192.168.1.1/24 interface=ether1 comment="LAN"
/ip address add address=172.18.0.1/16 interface=ether2 comment="WAN"
/ip route add gateway=x.x.x.x
/ping x.x.x.x
/ping google.com # verify DNS request
```

# Set up SSL certificate (assuming you have an existing cert)
```sh
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
```

# User audit: Create a new user and disable admin/root
```sh
/user print
/user set <idx or username> password="password"
/user set <idx or username> address=x.x.x.x/yy
/user add name=secureuser password="NewUserPass" group=full
/user set admin disabled=yes or /user disable admin
```

# Restrict SSL access to a specific IP
```sh
/ip firewall filter add chain=input protocol=tcp dst-port=443 src-address=192.168.1.100 action=accept comment="Allow SSL from specific IP"
/ip firewall filter add chain=input protocol=tcp dst-port=443 action=drop comment="Block all other SSL access"
```


## Harden the machine further (adjust as needed)

# MAC Connectivity Access
```sh
/interface list add name=LAN
/interface list member add list=LAN interface=____
/tool mac-server set allowed-interface-list=LAN
/tool mac-server mac-winbox set allowed-interface-list=LAN

# Do the same in the MAC Winbox Server tab to block Mac Winbox connections from the internet.
/tool mac-server mac-winbox set allowed-interface-list=none
/interface list add name=LAN
/interface list member add list=LAN interface=_______
/tool mac-server mac-winbox set allowed-interface-list=LAN
```
# Disable Neighbor Discovery & IP Connectivity Access
```sh
/ip neighbor discovery-settings set discover-interface-list=LAN
/ip firewall filter
  add chain=input action=accept connection-state=established,related,untracked comment="accept established,related,untracked"
  add chain=input action=drop connection-state=invalid comment="drop invalid"
  add chain=input in-interface=ether1 action=accept protocol=icmp comment="accept ICMP"
  add chain=input in-interface=ether1 action=accept protocol=tcp port=8291 comment="allow Winbox";
  add chain=input in-interface=ether1 action=drop comment="block everything else";
```

```sh
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
```
