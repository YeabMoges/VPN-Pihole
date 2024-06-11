# VPN-Pihole

## Update system & Install Docker

Updates the system to the latest package.
```sh
sudo apt update $$ sudo apt upgrade -y
```

Downlaod and install docker
```sh
sudo apt-get install docker -y
```
## Setup Pihole container

Create a new folder, in this case pihole
```sh
mkdir pihole
```

Make a docker-compose file
```sh
sudo nano docker-compose.yml
```

Copy docker-compose.yml.example to docker-compose.yml and update as needed
```sh
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"
    environment:
      TZ: 'America/Los_Angeles'
      WEBPASSWORD: 'your_password'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed
    restart: unless-stopped
```

Run `sudo docker-compose up -d` to build and start pi-hole (Syntax may be docker compose on some systems)
