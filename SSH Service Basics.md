ssh - port 22

systemctl status ssh (check status)
cd /etc/ssh
ls (can see all folders with ssh)
nano /etc/ssh/ssh_config (check some of the config files)
(include statement means this config files does not work by itself, it includes another config files as well)

nano /etc/ssh/sshd_config (another file that is great to look at)

can change ssh to different port if needed
ListenAddress - can change it to listen to just one IP address or just any IP address *very important*
PermitRootLogin - can allow root user to login or not, can change to <no>
(make sure to check all config folders and so forth)

ssh <username>@ip_address - use to connect to different terminals. 

_________________________________________________________________________________________
**helpful for generating a new key if compromised**
sudo nano ssh_host_rsa_key (check the keys for ssh files)
sudo nano ssh_host_rsa_key.pub (could use vi to format better)

cd .ssh (get into the secure ssh files)
ls (can see a known hosts files), cat <known_hosts>
sudo ssh-keygen -t <ecdsa> -f /etc/ssh/<whatever_is_compromised/key_file> (generate a new key)

___________________________________________________________________________________________
**setting up passwordless authentication for SSH** - use keys instead of password
sudo ssh.keygen -t <type_keys> ~/<id_bob_key>
ls /home/<name> -a (there might be no ssh files for him)
sudo mkdir /home/<name>/.ssh (this creates it but the name might not own it)
sudo cp <keys_file> /<new_file> (copy public key to users files)
la -la to check permissions

