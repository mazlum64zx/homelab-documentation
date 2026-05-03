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

   *
   * 

