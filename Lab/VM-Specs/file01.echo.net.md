# Yum repository and NFS/SMB File Server

This server is responsible for serving files to other nodes.  This includes NFS, SMB and a http yum repository.  This server only has one disk because it will be configured to use the iSCSI server as a backing store.

## VM Configuration

- CPU: 2
- RAM: 2048
- Disk: 10GB 
- Network: Infrastructure

## Network Configuration

- IPv4: 192.168.11.5/24
- GATE: 192.168.11.2
- DNS : 192.168.11.3
- Hostname: `file01.echo.net`
- Search Domain: `echo.net`

## Install

- Softwre Selection: Minimal
- Installation Destination: Automatic partitioning
- Security Policy: No profile
