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

