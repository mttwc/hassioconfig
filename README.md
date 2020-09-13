# HassioConfig

These instructions are for installing Home Assistant (Supervised) on an Intel NUC. By doing this, you have the flexibility of having HA add-ons, as well as better control over the host OS. You can also install other containers to run alongside HA.

## Why Supervised?

https://www.home-assistant.io/blog/2020/05/26/installation-methods-and-community-guides-wiki/

TLDR:

- Home Assistant full installation is a full OS installation, and you won't be able to run non-HA functions on the same machine
- Home Assistant Core does not support add-ons, and you need to manually configure other containers to get the same functionality
- Home Assistant Supervised supports add-ons while allowing you to run other containers in parallel for other functions you want to use the NUC for

## Installation instructions

Instructions are adapted from https://medium.com/@jordanrounds/intel-nuc-home-assistant-supervised-7cc52d81744a with some changes.

### Steps

1. Update the Intel NUC bios to latest

2. Follow the instructions at https://ubuntu.com/download/intel-nuc-desktop to install Ubuntu on the NUC (I used Ubuntu 16.04 LTS)
    - Note that when installing Ubuntu from USB, you will see a recovery prompt at the beginning. This is normal.

3. Assign a static IP. This will allow you to access Portainer from anywhere in the local network.

4. Install Docker

```bash
# Install depedencies
sudo apt-get update
sudo apt-get install \
  apt-transport-https \
  ca-certificate \
  curl \
  gnupg-agent \
  software-properties-common

# Install Docker's official GPG key and verify the fingerprint
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88

# Setup the stable repository
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

5. Install Portainer

```bash
# Allows you to navigate to Portainer in the local network with `<static IP>:9000`
docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

# Set Portainer to start automatically
docker ps # Identify the name of the Portainer container
docker run --restart=always <container-name>
```

  - Navigate to `<static IP>:9000` (or if locally, `localhost:9000`)
  - Set login credentials
  - Select Docker environment `Local`

6. Install Home Assistant Supervised

```bash
sudo su

add-apt-repository universe

apt-get install bash jq curl avahi-daemon dbus software-properties-common apparmor-utils apt-transport-https ca-certificates network-manager socat

systemctl disable ModemManager
systemctl stop ModemManager

# -m is the machine type (specific to Intel NUC)
# -d is saying we want to set the default data storage location
curl -sL https://raw.githubusercontent.com/home-assistant/supervised-installer/master/installer.sh | bash -s -- -m intel-nuc -d /home/docker/hassio

# https://github.com/home-assistant/supervised-installer
```

7. You can now navigate to Home Assistant on `<static IP>:8123`!

### Notes

- Do not enable firewall and ssf with `ufw`, as you can access the HA CLI through Portainer remotely. The firewall instructions in the medium article will disable connections to http://supervisor/core, and incoming remote connections, making DuckDNS and many add-ons break.

- If you are planning on using AdGuard Home or some other DNS ad-blocker, you will need to disable `systemd-resolved` and `dnsmasq` to free up port 53:

```bash
# Disable and stop systemd-resolved service
sudo systemctl disable systemd-resolved.service
sudo systemctl stop systemd-resolved

sudo su
vi /etc/NetworkManager/NetworkManager.conf
# Comment out `dns=dnsmasq` line with `#`

# --REBOOT--

# You should see this as cleared now, or used by Adguard if you've installed that add-on already
sudo lsof -i :53
```

- If you are planning on using Node-RED, connections to Home Assistant entities will likely fail unless you do the following:
    - In Node-RED, click any node
    - Hit the `Edit` button next to the `Server` dropdown
    - Uncheck `Delay connection attempts`

## Troubleshooting

### Getting systemd logs

Run the following command on a terminal in the host OS. These are low level logs that will show inner exception traces if HA Supervisor logs are not enough.
```
journalctl -f
```

