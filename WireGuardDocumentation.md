I started by signing up for digital ocean using the link provided. I then selected droplet as the project I want to make and put in the payment info so I could get started. After that I created a droplet based out of datacenter 3 in new york that used ubuntu version 24.04. I set it to use the basic plan with a regular intel cpu and normal ssd and gave it a password for the authentication method.

I opened the console through the website and followed part of the documentation I made earlier in order to install docker which I will have below:

-   Started by running this command in case any conflicting packages were installed: sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
    
-   I then set up the doctors apt repository first by updating the system using sudo apt update. Then running all of these commands to install dockers official GPG key.
    

sudo apt install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

-   I then added the repository to apt first through the command “sudo tee /etc/apt/sources.list.d/docker.sources <<EOF” then typing out each of these lines:
    

Types: deb

URIs: https://download.docker.com/linux/ubuntu

Suites: '$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")'

Components: stable

Signed-By: /etc/apt/keyrings/docker.asc

EOF

-   After this I used “sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin” to install docker and “sudo systemctl status docker” and “sudo docker run hello-world” to verify that it was installed and working properly.
Now that docker has been installed using my previous documentation it is time to install wireguard.

-   I started by making a directory for wireguard using “sudo mkdir -p /opt/stacks/wireguard” then changing to that directory with “cd /opt/stacks/wireguard”
    
-   Then I ran “sudo nano compose.yaml” to write a compose file and put in these lines:
    

version: '3.8'

services:

wireguard:

image: linuxserver/wireguard

container_name: wireguard

environment:

- PUID=1000

- PGID=1000

- TZ=America/Chicago

- SERVERURL=104.131.175.168

- SERVERPORT=51820

- PEERS=1

- PEERDNS=1.1.1.1

- ALLOWEDIPS=0.0.0.0/0

volumes:

- ./config:/config

- /lib/modules:/lib/modules

ports:

- "51820:51820/udp"

- "51821:51821/tcp"

restart: unless-stopped

cap_add:

- NET_ADMIN

- SYS_MODULE

sysctls:

- net.ipv4.ip_forward=1

- net.ipv4.conf.all.src_valid_mark=1
After creating the compose file I used “docker compose up -d” to get it running and then I began setting up the clients.

-   I started by running ls /opt/stacks/wireguard/config and saw the peer1 directory. I then used cat /opt/stacks/wireguard/config/peer1/peer1.conf to preview the file.
    
-   I used “sudo apt install wireguard” to install wireguard.
    
-   I then copied the config file to /ect/wireguard using “sudo cp /opt/stacks/wireguard/config/peer1/peer1.conf /etc/wireguard/wg0.conf” and “sudo chmod 600 /etc/wireguard/wg0.conf”
    
-   When I originally tried to bring up vpn using sudo wg-quick up wg-client I had an error so I went to the terminal on my actual laptop and used “scp root@104.131.175.168:/opt/stacks/wireguard/config/peer1/peer1.conf .” to download the peer to my laptop. Then I used sudo nano /etc/wireguard/wg-client.conf and added
    

[Interface]

Address = 10.20.20.2/32

PrivateKey = CKmGmWFa/T4nm1dhMVyI2bb2v+CYZ78mH48CEVE/DGw=

DNS = 1.1.1.1

  

[Peer]

PublicKey = Aw0R7ooq+IJGG79xkLqW9uLw1F9gVwHX5wz1s/AnZjA=

PresharedKey = TMnvReBlRiBJEcDlj5kdtMWF7GoYEYeYlKqtZwwCTok=

Endpoint = 104.131.175.168:51820

AllowedIPs = 10.20.20.0/24 # originally lazily put in 0.0.0.0/0 which was a bad idea for reasons I'll explain later.

  

-   After this I used sudo wg-quick up wg0 to bring up the vpn. Got it running successfully but as I routed all traffic through the vpn I couldn't access the terminal. To fix this I went to the droplets recovery console and ran sudo wg-quick down wg-client. After that I changed AllowedIPs = 10.20.20.0/24 to make sure this didn’t happen again.
    
-   I then used sudo wg-quick up wg0 to bring up the vpn, wg to check its status and sudo wg-quick down wg-client to bring it back down.
Now that I have the vpn running I need to test it both on my phone and laptop.

-   Start by turning the vpn back on. Then on windows powershell I typed scp root@104.131.175.168:/opt/stacks/wireguard/config/peer1/peer1.conf . to copy the file to my laptop.
    
-   I then installed the wireguard app on my phone through the appstore and on my laptop through this link [https://www.wireguard.com/install/](https://www.wireguard.com/install/)
    
-   To get it running on my phone I started by emailing the file to myself. Then I clicked the + icon on the app, chose create from file or archive, and selected the file. After that the vpn was set up on my phone and I was able to use [ipleak.net](http://ipleak.net) to verify that it worked.
