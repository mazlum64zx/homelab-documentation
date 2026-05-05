# Ubuntu Server Setup

## Goal

Set up a Linux server on an old laptop to learn the basics of system administration, networking, and server management.

The long-term goal is to experiment with self-hosted services such as:

* cloud storage (e.g. a personal Dropbox alternative)
* network-wide ad blocking
* other home lab applications

## Hardware

* Old laptop
* USB flash drive
* Main PC (Windows 11)

## Operating System

* Ubuntu Server LTS

## Current Progress

* Created GitHub account ✅
* documenting my progress
* Downloaded Ubuntu Server ISO ✅
* Prepared USB installation ✅
* Creating a bootalbe USB-Drive using Rufus
    * checking whether I need GPT or MBR on my older laptop in windows-> disk management -> disk 0 -> properties -> volumes ->       Partition stlye: GPT (in my case) ✅
* Installed Ubuntu Server ✅
   * wlan configuration -> DHCPv4 for now -> automatic ip address
* first time login using username + password
* to ensure consistent network access, a static IP address was assigned to the server using the router's DHCP reservation        feature. the router maps the server's MAC address to a fixed IP address, ensuring that the server always receives the same     IP when connecting to the network.
* using the command "sudo nano /etc/systemd/login.conf"  -> go to HandleLidSwitch=suspend and delete the "#" also from the
     HandleLidSwitchExternalPower=suspend, also HandleLidSwitch=Docked=ignore, as well as LidlSwitchIgnoreInhibited=yes.
  Then we change HandleLidSwitch=suspend to HandleLidSwitch=ignore, HandleLidSwitchExternalPower=suspend to    HandleLidSwitchExternalPower=ignore and LidlSwitchIgnoreInhibited=yes to -> LidlSwitchIgnoreInhibited=no
     * What we just did is to ensure, that the laptop is still being powered on even though we close the lit.
* connection in win11 terminal via ssh is failed -> the problem was my server didnt have an IPv4 Address asigned.
  -> fix via command "sudo apt install isc-dhcp-client", if I use "ip a" I know get a dedicated IPv4 and the
     connection via terminal to my server is now successful
* after sudo reboot the IPv4 is gone again and I can't connect to my server again -> trying to research a permanent fix
  ->This was caused by an incorrect Netplan configuration:
      dhcp4: true was placed in the wrong section (under WiFi or globally)
      YAML indentation was invalid
      As a result, DHCP was not applied on boot
*The fix was to disable the Wi-Fi configuration and ensure DHCP was correctly set for the Ethernet interface.
   Key change:
   ethernets:  enp0s31f6:    dhcp4: true

   Result
   After applying the corrected configuration, the system successfully obtained an IPv4 address on boot, and SSH               connections worked normally.

* installed tailscale on linux server and laptop for remote use
* installed casaOS -> connection succesful
* installed jellyfin -> cannot access server. -> checking sudo docker ps in terminal for which container are currently on
     -> starting casaOS via serverIP:8097(Port) instead of only ServerIP and now I can access JellyFinn
* installing pi hole -> not working because port 53 isn't available
     checking if my server already uses port 53 -> "sudo lsof -i :53"
     Port 53 can only be bound by one service at a time. On this system, systemd-resolved is already listening on port 53,       which prevents Pi-hole from starting its DNS service.
     free port 53 and keep DNS working):
      Stop and disable systemd-resolved (it occupies port 53), then replace the dynamic resolver config with a static DNS       server so the system keeps internet access.
      sudo cp /etc/resolv.conf /etc/resolv.conf.backup
      sudo systemctl stop systemd-resolved
      sudo systemctl disable systemd-resolved
      sudo rm /etc/resolv.conf
      echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
     Client & Pi-hole configuration:
      Disabled IPv6 on the client to prevent DNS bypass and manually set the IPv4 DNS server to the Pi-hole IP                    (192.168.0.216). Flushed and renewed the DNS configuration to apply changes. Additionally, configured the Pi-hole           container with FTLCONF_dns_listeningMode=all to allow DNS queries from the local network.
  * I initially planned to use Tailscale for remote access, but this was not possible due to missing administrator                privileges on my training laptop, which prevented me from installing the software. As an alternative, I switched to         Cloudflare. I created an account and purchased a domain for $5. The goal is to set up remote access to my server            through Cloudflare, allowing me to connect from my training laptop and continue working on my server during                 free time. At this point, I am in the setup phase of this configuration.

       The first step is to create a tunnel. Navigate in the Cloudflare dashboard to Zero Trust → Networks → Overview →            Manage Tunnels → Add a Tunnel (Create new cloudflared Tunnel)
          ->selecting operating system: debian , 64bit
          ->installing cloud flare on my server via command listed on the website and storing my token carefully.
          ->In the Cloudflare Zero Trust dashboard, I created a new cloudflared tunnel and configured a public hostname. I             assigned a subdomain and linked it to my domain. For the service configuration, I selected SSH and pointed it to            localhost:22, which connects the tunnel directly to the SSH service running on my server. This setup allows                 secure remote access to the server through Cloudflare.
          *In the Cloudflare Zero Trust dashboard, I created a self-hosted application for SSH access. I configured a                  public hostname using a subdomain (e.g., ssh.mazlum.uk) and enabled browser-based SSH rendering. I then created             an access policy to allow only my email address. After saving the configuration, I was able to securely access              my server via SSH directly through the browser using the Cloudflare tunnel.
    *After successfully setting up SSH access through Cloudflare, the next step was to access additional applications and        web interfaces running in my local network. These services are hosted on my server with the local IP address                192.168.0.216.

      Since direct access to a private IP address from outside the network is not possible, access had to be configured            through Cloudflare.

      To achieve this, I used the existing tunnel in the Cloudflare Zero Trust dashboard. I navigated to Networks →                Tunnels,       selected my tunnel, and then created a new route under Public Hostnames (Published application                routes).

      I configured the application as follows:

       Subdomain: casa.mazlum.uk
       Service type: HTTP
      Target address: localhost:80

      Since the CasaOS service is running directly on my server and is accessible via port 80, localhost in this context          refers to the same machine where the Cloudflare Tunnel (cloudflared) is running.

      After saving the configuration, the web interface was successfully accessible via:
      https://casa.mazlum.uk

   *With the Cloudflare Tunnel configured, I was now able to access my server’s web interface from any device with an internet connection, regardless of the network I was in. This means I could manage files and share content through the web interface without being limited to my local network.

After successfully accessing the CasaOS web interface via my domain, I encountered the next issue. When I clicked on installed applications such as Plex, I received a DNS error.

This behavior is expected: by default, applications like Plex are configured to use the server’s local IP address (e.g., 192.168.0.216) and a specific port (e.g., 32400). While this works within the local network, it fails when accessing the service remotely, since private IP addresses are not reachable from outside the network.

To resolve this, I created a new application route in Cloudflare. Under Networks → Tunnels → Public Hostnames, I added a new route with the following configuration:

Subdomain: plex.mazlum.uk
Service type: HTTP
Target address: http://localhost:32400

This configuration allows Cloudflare to route external requests for plex.mazlum.uk through the tunnel directly to the Plex service running on the server.

After that, I updated the Plex Web UI settings to use plex.mazlum.uk instead of the local IP address and removed the port from the URL.

This is necessary because Cloudflare only exposes standard HTTP/HTTPS ports (such as 80 and 443) to the public. If a custom port (e.g., 32400) is included in the URL, the browser will attempt to establish a direct connection to that port over the internet. Since this port is not supported or exposed by Cloudflare, the request will fail and will not be routed through the tunnel. By removing the port from the URL, all traffic is sent through Cloudflare on standard ports and then internally forwarded to the correct service via the tunnel.
             
     
     


## Installation

Ubuntu Server was installed using a bootable USB stick.

### side notes

Loopback Interface

The `lo` interface is the local loopback interface. It is used by the system to communicate with itself.

- IPv4 loopback address: `127.0.0.1`
- IPv6 loopback address: `::1`
- This interface is not used for external network access.
- ## Network Interfaces Overview

The `ip a` command was used to analyze the network interfaces.

### Interfaces

* `lo` – loopback interface (internal communication)
* `enp0s31f6` – Ethernet (LAN)
* `wlp02s0f3` – WiFi interface

### Key Information

* Server IP address: `192.168.0.194`
* Assigned dynamically via DHCP
* MAC addresses:

  * LAN: `38:f3:ab:b5:2a:fb`
  * WiFi: `ec:63:d7:91:6b:ad`

### Result

The server is successfully connected to the network via WiFi and is reachable using its assigned IP address.


## Notes / Problems

   *cannot use file sharing in casa -> trying to install samba with sudo apt install samba -y -> samba already installed
   * 

