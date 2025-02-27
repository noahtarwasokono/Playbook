ssh - port 22

# Basic SSH things to look at 
```sh
systemctl status ssh (check status)
cd /etc/ssh
ls (can see all folders with ssh)
nano /etc/ssh/ssh_config (check some of the config files)
(include statement means this config files does not work by itself, it includes another config files as well)

nano /etc/ssh/sshd_config (another file that is great to look at)

# can change ssh to different port if needed
# ListenAddress - can change it to listen to just one IP address or just any IP address *very important*
# PermitRootLogin - can allow root user to login or not, can change to <no>
(make sure to check all config folders and so forth)

# ssh <username>@ip_address - use to connect to different terminals. 
```
_________________________________________________________________________________________
**helpful for generating a new key if compromised**
```sh
sudo nano ssh_host_rsa_key (check the keys for ssh files)
sudo nano ssh_host_rsa_key.pub (could use vi to format better)

cd .ssh (get into the secure ssh files)
ls (can see a known hosts files), cat <known_hosts>
sudo ssh-keygen -t <ecdsa> -f /etc/ssh/<whatever_is_compromised/key_file> (generate a new key)
```

___________________________________________________________________________________________
**setting up passwordless authentication for SSH** - use keys instead of password
```sh
sudo ssh.keygen -t <type_keys> ~/<id_bob_key>
ls /home/<name> -a (there might be no ssh files for him)
sudo mkdir /home/<name>/.ssh (this creates it but the name might not own it)
sudo cp <keys_file> /<new_file> (copy public key to users files)
la -la to check permissions
```

Config files:
- `/etc/ssh/sshd_config`
	- Port 22
	- PermitRootLogin : allows people to login as the root user over ssh, usually disable
	- Max Sessions : set a max number of users, limit it to you and scoring
	- AuthorizedKeysFile : lets you select a location to add key pairs (two by default, lock it down). Red team likes to either add more to the one line or add a second line with their custom
	- Permit Empty Passwords: 
	- PasswordAuthentication : once you have keys enabled, just turn off password authentication, check ownership and permissions, too
	- *** `UsePAM yes` *** : this lets you authenticate
```sh
LoginGraceTime 2m
PermitRootLogin no
MaxAuthTries 6
MaxSessions 5

PubkeyAuthentication yes

AuthorizedKeysFile ...

PasswordAuthentication no # this also stops key copy
PermitEmptyPasswords no
```
- `/etc/ssh/sshd_config.d/05-cloud-init.conf`
	- PasswordAuthentication : you need this off here and in `sshd_config` to turn off password authentication
For a user, `~/.ssh/authorized_keys`:
- Public keys go here
- Check all authorized users for correct keys
- Back up good keys in a safe location
- Probably back up `/etc/shadow`, too for passwords

You can generate ssh keys using the `ssh-keygen` command:
	You'll need to specify a keyfile
	-t : type of key, ex rsa or ecdsa
	-f : file location, you can overwrite the existing keys and probably should if your computer is a clone
Copy the public key to the remote host:
`ssh-copy-ip -i <key.pub> user@ip.ad.dr.ess`
To connect: 
`ssh -i <key> user@ip.ad.dr.ess` 

`sudo apt install openssh-server`
`systemctl status ssh` or `systemctl status sshd`

Permissions for config files:
```
-rw-r--r-- root root ssh_config
-rw-r--r-- root root sshd_config
-rw------- root root ssh_key
-rw-r--r-- root root ssh_key.pub
```
