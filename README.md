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

### Create a directory 
In this case pihole
```sh
mkdir pihole
```

### Make a docker-compose file
```sh
sudo nano docker-compose.yml
```

Copy docker-compose.yml.example to docker-compose.yml and update as needed. 
Refer [docker-pi-hole](https://github.com/pi-hole/docker-pi-hole?tab=readme-ov-file) for customization
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
      # WEBPASSWORD: 'set a secure password here or it will be random, uncomment to edit
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

Make sure pihole is up by running `sudo docker-compose ps`

Document pihole container IP address by running
```sh
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' pihole
```


## Setup Wireguard container
 
### Create a wireguard directory
```sh
mkdir wireguard
```

### Make a docker-compose file
```sh
sudo nano docker-compose.yml
```

Copy docker-compose.yml.example to docker-compose.yml and update as needed. 
Refer [wg-easy](https://github.com/wg-easy/wg-easy/tree/master) for customization
```sh
version: "3.8"

services:
  wg-easy:
    container_name: wireguard
    image: ghcr.io/wg-easy/wg-easy
    environment:
      - PASSWORD=your_password
      - WG_HOST=your_IP_Address
      - WG_DEFAULT_DNS=Pihole_IP_Adrress
      - WG_PORT=51820
    volumes:
      - ./config:/etc/wireguard
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
```
Replace
`your_password` to your new secure wireguard Web Interface password. 

`your_IP_Address` with your public Ip address, run ```sh curl ifconfig.me ``` 

`Pihole_IP_Adrress` with the documented pihole address

Run `sudo docker-compose up -d` to build and start wireguard contaienr

Make sure wireguard is up by running `sudo docker-compose ps`

### Connect the WireGuard container to the same network as the Pi-hole container.

```sh
docker network connect pihole_default wireguard
```


## Port Forwarding for WireGuard

To enable WireGuard, you need to configure port forwarding on your router. Since the settings vary depending on the router model, please refer to this [guide on opening ports](https://nordvpn.com/blog/open-ports-on-router/) for detailed instructions.

### Steps to Follow

1. Access your router's configuration page.
2. Locate the port forwarding section in the router settings.
3. Select your device running the WireGuard server.
4. Forward the following port:
   - **Port:** 51820
   - **Protocol:** UDP

## Access Pihole and Wireguard Web Interface

### Geting your Device IP Address
Run the following command
```sh
hostname -I
```
### Accessing the Web Interfaces
Open a web browser on a device that is connected to the same local network as the device running Pi-hole and WireGuard.

Navigate to `http://<Devices_IP_Address>/admin` to access Pihole Web

Navigate to `http://<Devices_IP_Address>:51821` to access Wireguard Web 

Replace `<Device_IP_Address>` with the IP address obtained from the `hostname -I` command.























