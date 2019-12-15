# IPA Server for LDAP, Kerberos, DNS, and NTP

This server is responsible for providing NTP, LDAP Authentication, and Kerberos Authorization.  This includes authoritative DNS over the `echo.net` doamin.

## VM Configuration

- CPU: 2
- RAM: 4096
- Disk: 15GB 
- Network: Infrastructure

## Network Configuration

- IPv4: 192.168.11.3/24
- GATE: 192.168.11.2
- DNS : 127.0.0.1
- Hostname: `ipa01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
