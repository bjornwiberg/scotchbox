## Vagrant box based on scotchbox 2.5
* Allows multiple sites on same container

* External storage for site files, logs and database

* Support for both private and public networks via simple terminal command

### To clone project
```
git clone https://github.com/bjornwiberg/scotchbox.git ~/scotch
```
### To start box
```
cd ~/scotch/
vagrant up
```

### Trust the root certificate for https usage on sites
#### Mac
```
cd ~/scotch/ && sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ssl/certs/ca.cert.pem
```
#### Other oses
```
install cert and trust in ```ssl/certs/certs/ca.cert.pem```
```
### Install scripts for easy site creation
source file into your config e.g. .zshrc
```
echo "source ~/scotch/scotchfunctions" >> ~/.zshrc && source ~/.zshrc
```
### To add new sites to box
use command newsite

this command add hostname to Domains, add hostname to /etc/hosts and provision the box
```
newsite hostname.local
```

### Set correct network adapter when running public network mode
If your network adapter is not called Wi-Fi (AirPort) you need to make an extra step

When starting box an dilalog like this will be displayed
```
==> default: Specific bridge 'en0: Wi-Fi (AirPort)' not found. You may be asked to specify
==> default: which network to bridge to.
==> default: Available bridged network interfaces:
1) en0: Wi-Fi
2) p2p0
3) awdl0
4) en1: Thunderbolt 1
5) en2: Thunderbolt 2
```

Make your choice

#### Edit Vagrantfile on line 10
```config.vm.network "#{NETWORK_TYPE}", ip: "#{IP}", bridge: "en0: Wi-Fi (AirPort)"```

replace ```en0: Wi-Fi (AirPort)``` with the choice you make in last step e.g. ```en1: Thunderbolt 1```

line 10 will now look like this

```config.vm.network "#{NETWORK_TYPE}", ip: "#{IP}", bridge: "en1: Thunderbolt 1"```

Reload the box
```
vagrant reload
```

### To change from public to private network
This will update the default network config and update ```/etc/hosts```

#### Private
```
ipprivate
```
#### Public
Edit ```~/scotch/networkconfig/nw_public``` with your desired ip, remember to update you functions ippublic and ipprivate with same ip

```
ippublic
```
