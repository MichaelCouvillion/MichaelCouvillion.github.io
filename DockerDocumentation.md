Downloading Docker and Docker Compose on Ubuntu

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
After Installing It was time to set up an application. For this I chose Gotify, an application that's used for receiving notifications.

-   I started by making and switching to a directory for gotify using the commands “mkdir -p ~/gotify” and “cd ~/gotify”.
    
-   I then created the docker compose file using the command “nano docker-compose.yml” and added the following to the file.
    

services:

gotify:

image: gotify/server:latest

container_name: gotify

restart: unless-stopped

ports:

- "80:80"

volumes:

- ./data:/app/data

-   Before starting the application it's important to run “sudo usermod -aG docker $USER” to give docker sudo permissions as I errors because I forgot this step.
    
-   Use the command “docker compose up -d” to create the application, “docker compose ps” to check the status and “docker compose logs -f” to see the logs.
-   I then restarted gotify using “docker compose down” and “docker compose up -d”
    
-   Find your VM ip using “ip a” and it’ll be next to inet.
    
-   After visiting [http://192.168.134.129](http://192.168.134.129) I was prompted to log in. I then went to the Applications tab, clicked create application, called it app and copied the token it created for me.
    
-   To send a test notification I used the command:
    

curl -X POST "http://192.168.134.129/message?token=AtiMZlgS.bYX6gd" \

-F "title=Message" \

-F "message=It works"\

  

After running this the site successfully got the message showing that things were set up correctly.
