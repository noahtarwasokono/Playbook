### Here are step-by-step instructions with commands for forcing all web traffic through a proxy server to isolate internal workstations and servers, for both Linux and Windows environments.

### Linux: Transparent Proxy with Squid + Iptables

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
Configure internal workstations' default gateway to point to this proxy/firewall machine.


### Windows: Proxy and Firewall Setup

##### Step 1: Set System Proxy Settings via PowerShell or Group Policy
For manual PowerShell (run as Administrator), replace proxy address and port (e.g., 192.168.1.10:3128):
```sh
netsh winhttp set proxy 192.168.1.10:3128
```

Or via Windows GUI
- Settings > Network & Internet > Proxy > Use a proxy server (enable and set IP and port)
- For domain environments, use Group Policy under: User Configuration > Policies > Windows Settings > Internet Explorer Maintenance > Connection > Proxy Settings

##### Step 2: Block Direct Internet Access via Windows Firewall
Open elevated PowerShell and run:
```sh
# Block outbound HTTP and HTTPS traffic except from proxy server
New-NetFirewallRule -DisplayName "Block HTTP outbound" -Direction Outbound -Protocol TCP -RemotePort 80 -Action Block
New-NetFirewallRule -DisplayName "Block HTTPS outbound" -Direction Outbound -Protocol TCP -RemotePort 443 -Action Block
New-NetFirewallRule -DisplayName "Allow Proxy Server" -Direction Outbound -RemoteAddress 192.168.1.10 -Action Allow
```
