# iSCSI Target

This server is responsible for providing iSCSI Storage.

## VM Configuration

- CPU: 2
- RAM: 2048
- Disk: 10GB 
- Disk1: 1GB (NFS)
- Disk2: 1GB (SMB)
- Disk3: 1GB (DB)
- Network: Infrastructure

## Network Configuration

- IPv4: 192.168.11.6/24
- GATE: 192.168.11.2
- DNS : 192.168.11.3
- Hostname: `iscsi01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
