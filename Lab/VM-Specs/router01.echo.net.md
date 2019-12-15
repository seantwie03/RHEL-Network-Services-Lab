# Router-NetworkTeaming

This server is responsible for routing traffic between the DMZ and Infrastructure subnets.  It also has multiple virtual NICs on each network to support teaming.

## VM Configuration

- CPU: 2
- RAM: 2048
- Disk: 10GB 
- Network1: DMZ
- Network2: DMZ
- Network3: Infrastructure
- Network4: Infrastructure

## Network Configuration

- Team0 (DMZ):
    - Network1 & Network2
    - IPv4: 192.168.10.2/24
    - DNS : 192.168.11.3

- Team1 (Infrastructure):
    - Network3 & Network4
    - IPv4: 192.168.11.2/24
    - DNS : 192.168.11.3

- Hostname: `router01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
