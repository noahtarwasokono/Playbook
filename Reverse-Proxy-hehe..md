## Here’s a practical, step-by-step guide for setting up an Active Defense Reverse Proxy with Nginx/OpenResty on both Linux and Windows, suitable for intercepting and disrupting attacker C2 traffic in a competition environment.

### Linux Setup
1. Install OpenResty (Nginx with Lua scripting)
OpenResty extends Nginx with Lua scripting for traffic manipulation.

```sh
sudo apt update
sudo apt install -y curl gnupg2
curl -O https://openresty.org/package/pubkey.gpg
sudo apt-key add pubkey.gpg
sudo apt update
sudo apt install -y openresty
```

2. Basic Nginx Reverse Proxy Setup with Lua Script
Edit /usr/local/openresty/nginx/conf/nginx.conf to add this Lua logic:

```sh
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    lua_shared_dict c2_delays 10m;

    server {
        listen 8080;

        location / {
            access_by_lua_block {
                local c2_delays = ngx.shared.c2_delays
                local ip = ngx.var.remote_addr

                local delay = c2_delays:get(ip) or 0

                if delay > 5 then
                    ngx.exit(ngx.HTTP_FORBIDDEN)
                elseif delay > 0 then
                    ngx.sleep(delay)
                    c2_delays:incr(ip, 1)
                else
                    c2_delays:set(ip, 1, 300)
                end
            }

            proxy_pass http://real-c2-server:port;  # Replace with actual C2 endpoint
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```
This script delays repeated requests from the same IP, then blocks.

Customize and extend for content inspection or payload tampering.

3. Start OpenResty
   
```sh
sudo systemctl start openresty
sudo systemctl enable openresty
```

4. Force attacker traffic to proxy
Use iptables or firewall rules to redirect traffic destined for C2 IP/ports to your proxy server on port 8080.

### Windows Setup (Using Docker + Nginx Proxy)

