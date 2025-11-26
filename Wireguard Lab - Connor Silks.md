**Preliminary Setup:**

To begin, I set up a Digital Ocean account and created a Ubuntu 24.04 droplet.
![[Pasted image 20251125232502.png]]

Following the previous project's steps, I installed docker on the droplet.

**Wireguard Setup:**

Following this video (https://www.youtube.com/watch?v=GZRTnP4lyuo) loosely, paired with previous knowledge of docker compose and the slides, I created a wireguard directory with ``mkdir -p ~/wireguard/`` and its config directory with ``mkdir -p ~/wireguard/config/`` then created a docker compose file with ``nano ~/wireguard/docker-compose.yml`` and input the following content to create the wireguard docker compose based on the video and slides:
```
    version: '3.8'
    services:
      wireguard:
        container_name: wireguard
        image: linuxserver/wireguard
        environment:
          - PUID=1000
          - PGID=1000
          - TZ=America/Chicago # changed to fit timezone
          - SERVERURL=129.212.181.190 # this is the Droplet's / Server's IP
          - SERVERPORT=51820
          - PEERS=pc1,pc2,phone1 # the connections and configs that are generated
          - PEERDNS=auto # auto setup DNS with them
          - INTERNAL_SUBNET=10.0.0.0 
        ports:
          - 51820:51820/udp
        volumes:
          - type: bind
            source: ./config/
            target: /config/
          - type: bind
            source: /lib/modules 
            target: /lib/modules
        restart: always
        cap_add:
          - NET_ADMIN
          - SYS_MODULE
        sysctls:
          - net.ipv4.conf.all.src_valid_mark=1
```

After saving the docker compose file, the server is created after running ``docker compose up -d`` after ``cd wireguard`` to enter the wireguard directory and then start the container with the compose setup.

To test the VPN, I installed the Wireguard App on both my phone and laptop. For the phone, I first visited IPLeak.net to see my initial local IP info before enabling the tunnel:
![[Pasted image 20251125233943.png]]

After enabling the VPN by scanning the QR code generated for the phone1 peer using ``docker logs wireguard`` to get the config for it and enabling it in the app, the IP addresses and info show the following:

![[Pasted image 20251125234340.png]]

As intended, the IP addresses are different after enabling the VPN tunnel.

For the laptop, I ran ``cat ~/wireguard/configs/peer_pc1/peer_pc1.conf`` and copied the contents into a .conf file I created on my laptop. Then, I imported the config into the Wireguard app on my laptop. Prior to starting the VPN tunnel, IPLeak.net looked like this:
![[Pasted image 20251125234714.png]]

After starting the VPN tunnel, it looked like this:
![[Pasted image 20251125230711.png]]

Finally, the image below demonstrates that the VPN tunnel was active on the app:
![[Pasted image 20251125230925.png]]