### Here are step-by-step instructions with commands for forcing all web traffic through a proxy server to isolate internal workstations and servers, for both Linux and Windows environments.

#### Linux: Transparent Proxy with Squid + Iptables

##### Step 1: Update System
```sh
sudo apt update -y
sudo apt upgrade -y
```

##### Step 2: Install Squid Proxy Server
```sh
sudo apt install squid -y
```

##### Step 3: Enable IP Forwarding
Edit /etc/sysctl.conf, uncomment or add:
```sh
net.ipv4.ip_forward = 1
sudo sysctl -p
```

##### Step 4: Configure Squid for Transparent Proxy
Edit Squid config file and modify/add lines:
```sh
sudo nano /etc/squid/squid.conf
http_port 3128 intercept
acl localnet src 192.168.1.0/24       # Replace with your LAN subnet
http_access allow localnet
http_access deny all
visible_hostname proxy-server
```

##### Step 5: Start and Enable Squid
```sh
sudo systemctl restart squid
sudo systemctl enable squid
```

###### Configure Firewall (iptables)
Assuming eth0 is internal LAN interface and eth1 is outbound interface:
Redirect HTTP traffic to Squid:
```sh
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3128
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
sudo iptables-save > /etc/iptables/rules.v4
```

Optional to block direct outbound except Squid
```sh
sudo iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 80 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 443 -j ACCEPT
# (Adjust as needed based on HTTPS proxy capability, for transparent HTTPS extra config is needed)

sudo iptables -A FORWARD -i eth0 -o eth1 -j DROP
```

##### Step 7: Set Client Default Gateway
