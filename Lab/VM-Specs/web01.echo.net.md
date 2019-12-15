# HTTP-HTTPS Web Server

This server serves web traffic.  The application is a basic shell script the queries the Database server and displays the results.  To re-enforce SELinux concepts the shell script is stored in a non-standard directory.

## VM Configuration

- CPU: 2
- RAM: 2048
- Disk: 10GB 
- Network: Infrastructure

## Network Configuration

- IPv4: 192.168.10.4/24
- GATE: 192.168.10.2
- DNS : 192.168.11.3
- Hostname: `web01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
