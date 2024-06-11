# VPN-Pi-hole

This guide will help you set up a VPN using WireGuard and Pi-hole for ad blocking on a Raspberry Pi. The VPN service allows you to connect to your local network remotely. By combining WireGuard and Pi-hole, you can ensure that all your network traffic is encrypted and filtered to block ads, providing a secure and ad-free browsing experience while at home and remotely.

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
cd pihole
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

Replace

  - `your_password` to your new secure pihole Web Interface password. 

Run the following command to build and start Pi-hole:
```sh 
sudo docker-compose up -d
```

Verify Pi-hole is running:
```sh
sudo docker-compose ps
```

Document pihole container IP address:
```sh
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' pihole
```


## Setup Wireguard VPN container
 
### Create a wireguard directory
```sh
mkdir wireguard
cd wireguard
```

### Create a docker-compose file
Create and edit the `docker-compose.yml` file.
```sh
sudo nano docker-compose.yml
```

Copy and paste the following configuration into the file and update as needed. 
Refer [wg-easy](https://github.com/wg-easy/wg-easy/tree/master) repository for customization options.
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

  - `your_password` to your new secure wireguard Web Interface password. 

  - `your_IP_Address` with your public Ip address, run `curl ifconfig.me` 

  - `Pihole_IP_Adrress` with the documented pihole address

Run the following command to build and start the WireGuard container:

```sh
sudo docker-compose up -d
```

Verify WireGuard is running: 
```sh
sudo docker-compose ps
```

### Connect the WireGuard container to the Pi-hole container.
Connect the WireGuard container to the same network as the Pi-hole container:
```sh
sudo docker network connect pihole_default wireguard
```

Verify connection by pinging Pi-hole container
```sh
sudo docker exec -it wireguard ping <Pihole_IP_Address>
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

## Additional Features

### Setting Pi-hole as DNS Server on Your Devices

  - On any of your devices, go to the network settings and set the DNS server to `Device_IP_Address`.

This will ensure that all DNS queries are routed through the Pi-hole, providing ad blocking and tracking capabilities even when not connected to the VPN.






