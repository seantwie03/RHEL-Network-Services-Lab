# SMTP-DNS

This server is the central relay for mail.  Other hosts send their email to this server.  It is also configure as a caching-only DNS server.

## VM Configuration

- CPU: 2
- RAM: 2048
- Disk: 10GB 
- Network: DMZ

## Network Configuration

- IPv4: 192.168.10.3/24
- GATE: 192.168.11.2
- DNS : 192.168.11.3
- Hostname: `smtp01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
