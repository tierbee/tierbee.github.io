---
title: title
---

**Docker Installation:** 

Following the Docker Engine Install Wiki, specifically in the Ubuntu section to align with my Linux Distribution of choice for my Proxmox VM, I began by checking if any conflicting packages needed to be uninstalled with ``sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)``. None of the packages ended up being installed. 

Using the following commands, the Docker apt repository is set up so that Docker's packages can be installed:
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Now, since Docker's repository is added to the apt sources, Docker's paclages can be installed through running ``sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin``. Then, I verified that Docker was running after the install by using ``sudo systemctl status docker``, which told me that Docker was active and the Docker installation was completed.

**Application Setup:**

I chose to install the Uptime Kuma application.

Using their guide to installation, https://uptimekuma.org/install-uptime-kuma-docker/, I started by creating a directory for Uptime Kuma and entered the directory after with ``mkdir -p ~/uptime-kuma && cd ~/uptime-kuma``. Then I created a docker-compose.yml file and opened an editing environment using ``nano docker-compose.yml``. The following code block is the content of my docker-compose.yml file after editing the template that Uptime Kuma provides in their guide to set a persistent storage location and edit the timezone: 
```
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: always
    ports:
      - "3001:3001"
    volumes:
      - ./uptime-kuma/kuma_data:/app/data
    environment:
      - TZ=America/Chicago
      - UMASK=0022  
    networks:
      - kuma_network  
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 5s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  kuma_network:
    driver: bridge
```

From there, all that is left is to run ``docker compose up -d`` to start the app's container and visit http://0.0.0.0:3001 to see the running application as shown below:
![[Pasted image 20251116221036.png]]

Below is an image of the container's information to find the server IP needed to know what URL to visit to find the running application:
![[Pasted image 20251116221106.png]]


