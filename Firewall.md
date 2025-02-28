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

```
