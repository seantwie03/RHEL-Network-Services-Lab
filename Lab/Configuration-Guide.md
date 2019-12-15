# RHEL Network Services Lab - Configuration
This document covers how to setup each node in the Network Services Lab.  Each VM in the lab has a file named after it, refer to those files for specific information about configuring the virtual machines.

For an overview of the Network Services Lab, including a short description of each node, please see the [RHEL Network Services Lab](../README.md#rhel-network-services-lab) section of the README.

*NOTE:* Unless stated otherwise, all commands must be ran as root and anything written in <THIS_NOTATION> needs to be changed to suite the environment.

- [RHEL Network Services Lab - Configuration](#rhel-network-services-lab---configuration)
  * [Create the `file01.echo.net` Virtual Machine](#create-the--file01echonet--virtual-machine)
  * [Setup a local repo on `file01.echo.net`](#setup-a-local-repo-on--file01echonet-)
  * [Setup Apache to share Echo.Net Repository](#setup-apache-to-share-echonet-repository)
  * [Create the `ipa01.echo.net` Virtual Machine](#create-the--ipa01echonet--virtual-machine)
  * [Setup IPA Server](#setup-ipa-server)
  * [Join `file01.echo.net` to the `ECHO.NET IPA Realm`](#join--file01echonet--to-the--echonet-ipa-realm-)
  * [Configure `file01.echo.net` to use `ipa01.echo.net` as the NTP Server](#configure--file01echonet--to-use--ipa01echonet--as-the-ntp-server)
  * [Create Virtual Machines `router01.echo.net` and `smtp01.echo.net`](#create-virtual-machines--router01echonet--and--smtp01echonet-)
  * [Configure the Teaming connection on the Infrastructure NICs of `router01.echo.net`](#configure-the-teaming-connection-on-the-infrastructure-nics-of--router01echonet-)
  * [Configure the Teaming connection on the DMZ NICs of `router01.echo.net`](#configure-the-teaming-connection-on-the-dmz-nics-of--router01echonet-)
  * [Enable `router01.echo.net` to route traffic between subnets](#enable--router01echonet--to-route-traffic-between-subnets)
  * [Disable firewalld on `router01.echo.net`](#disable-firewalld-on--router01echonet-)
  * [Verify `router01.echo.net` configuration after reboot](#verify--router01echonet--configuration-after-reboot)
  * [Common Configuraton of `router01.echo.net` and `smtp01.echo.net`](#common-configuraton-of--router01echonet--and--smtp01echonet-)
  * [Configure `smtp01.echo.net` to recieve mail from `@ECHO.NET`](#configure--smtp01echonet--to-recieve-mail-from---echonet-)
  * [Configure `ipa01.echo.net` and `file01.echo.net` to relay email to `smtp01.echo.net`](#configure--ipa01echonet--and--file01echonet--to-relay-email-to--smtp01echonet-)
  * [Create Virtual Machine `iscsi01.echo.net`](#create-virtual-machine--iscsi01echonet-)
  * [Perform common configuration steps on `iscsi01.echo.net`](#perform-common-configuration-steps-on--iscsi01echonet-)
  * [Configure iSCSI Target for NFS, SMB, and Database](#configure-iscsi-target-for-nfs--smb--and-database)
  * [Connect `file01.echo.net` to iSCSI Target `iscsi01.echo.net`](#connect--file01echonet--to-iscsi-target--iscsi01echonet-)
  * [Configure NFS Shares](#configure-nfs-shares)
  * [Mount NFS Shares on `ipa01.echo.net`](#mount-nfs-shares-on--ipa01echonet-)
  * [Configure SMB shares](#configure-smb-shares)
  * [Mount SMB Shares on `ipa01.echo.net`](#mount-smb-shares-on--ipa01echonet-)
  * [Create Virtual Machine `db01.echo.net`](#create-virtual-machine--db01echonet-)
  * [Perform common configuration steps on `db01.echo.net`](#perform-common-configuration-steps-on--db01echonet-)
  * [Configure MariaDB Database (Non-Standard Port)](#configure-mariadb-database--non-standard-port-)
  * [Create and Restore a MariaDB Backup](#create-and-restore-a-mariadb-backup)
  * [Create Virtual Machine `web01.echo.net`](#create-virtual-machine--web01echonet-)
  * [Perform common configuration steps on `web01.echo.net`](#perform-common-configuration-steps-on--web01echonet-)
  * [Install and Configure Apache on `web01.echo.net`](#install-and-configure-apache-on--web01echonet-)
  * [Add HTTP and HTTPS VirtualHost for `users.echo.net`](#add-http-and-https-virtualhost-for--usersechonet-)
  * [Create 'Mock' application to query database](#create--mock--application-to-query-database)


## Create the `file01.echo.net` Virtual Machine
Crate the `file01.echo.net` virtual machine as specified in its VM-Specs file:
- [file01.echo.net](VM-Specs/file01.echo.net.md)

## Setup a local repo on `file01.echo.net`

- Using Virtual Machine Manager (or similar) add the CentOS-7-DVD ISO to the `file01.echo.net` Virtual Machine

- Mount the ISO

      mount -t iso9660 /dev/sr0 /mnt

- Remove the default CENTOS Repos

      mv /etc/yum.repos.d/*.repo /tmp/

- Add CD-ROM yum repository

      vi /etc/yum.repos.d/cdrom.repo
      *Add the following lines:
      [CDRom]
      name=CDRom
      baseurl=file:///mnt
      enabled=1
      gpgcheck=1
      gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

- Verify the repo configuration is working by installing createrepo

      yum install -y yum-utils createrepo

- Create a directory for local repo

      mkdir -p /var/repo

- Sync CDRom repo to /var/repo

      reposync -r CDRom --downloadcomps --download-metadata --gpgcheck --norepopath -p /var/repo

- Create local repo metadata with groups

      createrepo /var/repo -g /var/repo/comps.xml

- Remove cdrom.repo

      mv /etc/yum.repos.d/cdrom.repo /tmp/

- Add local repo

      vi /etc/yum.repos.d/echo.net.repo
      *Add the following lines:
      [Echo.Net.Repo]
      name=Echo.Net Repo
      baseurl=file:///var/repo
      enabled=1
      gpgcheck=1
      gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

- Verify the local repo is working

      yum install -y vim

- Unmount the ISO

      umount /mnt

- Remove ISO from Virtual Machine


## Setup Apache to share Echo.Net Repository

- Login to `file01.echo.net`
- Install Apache

      yum install -y httpd

- Start and Enable httpd

      systemctl enable httpd --now

- Allow http and https through the firewall

      firewall-cmd --permanent --add-service=http
      firewall-cmd --reload

- Create symbolic link to DocumentRoot

      ln -s /var/repo /var/www/html/repo

- Install semanage command

      yum install -y policycoreutils-python

- Change SELinux context

      semanage fcontext -a -t httpd_sys_content_t "/var/repo(/.*)?"
      restorecon -Rv /var/repo

- Verify previous steps

      curl http://localhost/repo/

- Update repo config

      vi /etc/yum.repos.d/echo.net.repo
      *Change the following line
      FROM
      baseurl=file:///var/repo
      TO
      baseurl=http://localhost/repo/

- Verify repo configuration

      yum clean all && yum makecache


## Create the `ipa01.echo.net` Virtual Machine
Crate the `ipa01.echo.net` virtual machine as specified in its VM-Specs file:
- [ipa01.echo.net](VM-Specs/ipa01.echo.net.md)


## Setup IPA Server
- Login to `ipa01.echo.net`
- Add the `Echo.Net` Repo
    - Follow the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)

- Set the time

      hwclock --hctosys
      
- Verify time

      timedatectl

- Install the IPA Server

      yum install -y ipa-server ipa-server-dns

- Add FQDN to hosts file

      vi /etc/hosts
      *Add the following:
      192.168.11.3  ipa01.echo.net  ipa01

- Install IPA Server

      ipa-server-install --setup-dns
      Accept Defaults
      Enter passwords as prompted
      When asked to setup DNS forwarders, select No
      When asked to search for reverse zones, selct No
      When asked to configure system with these values, select yes
      
- Verify kerberos is working

      kinit admin
      klist

- Configure firewalld to allow IPA

      firewall-cmd --permanent --add-service http --add-service https --add-service kerberos --add-service dns --add-service ntp --add-service ldap --add-service ldaps
      firewall-cmd --reload

- Add reverse DNS zone for 192.168.11.0/24

      ipa dnszone-add --name-from-ip=192.168.11.0/24

- Add reverse DNS zone for 192.168.10.0/24

      ipa dnszone-add --name-from-ip=192.168.10.0/24

- Create PTR record for `ipa01.echo.net`

      ipa dnsrecord-add 11.168.192.in-addr.arpa. 3 --ptr-rec ipa01.echo.net

- Verify PTR record

      dig -x 192.168.11.3
      *Look for 'status: NOERROR'

- Allow PTR record syncronization from forward zone to reverse zones

      ipa dnszone-mod echo.net. --allow-sync-ptr=True
      ipa dnszone-mod 11.168.192.in-addr.arpa --dynamic-update=TRUE
      ipa dnszone-mod 10.168.192.in-addr.arpa --dynamic-update=TRUE

- Add a kerberos user - jdoe

      ipa user-add
      *Enter the following at the prompts
      First: John
      Last: Doe
      User: jdoe

- Add a kerberos user - tdoe

      ipa user-add
      *Enter the following at the prompts
      First: Tanya
      Last: Doe
      User: tdoe

- Create a kerberos group - doegroup

      ipa group-add doegroup

- Add jdoe to doegroup

      ipa group-add-member
      *Group name: doegroup
      *member user: jdoe
      *member group: [blank]

- Add tdoe to doegroup

      ipa group-add-member
      *Group name: doegroup
      *member user: tdoe
      *member group: [blank]

- Create temporary password for jdoe

      ipa user-mod jdoe --password
      *Enter a temp password -- you will have to change it upon login

- Create temporary password for tdoe

      ipa user-mod tdoe --password
      *Enter a temp password -- you will have to change it upon login

- Create password for jdoe

      ssh jdoe@localhost
      *Enter temp passowrd
      *Enter temp passowrd
      *Create new password
      *Confirm new password

- Verify a kerberos ticket was granted

      klist
      *Look for Default Principle jdoe@ECHO.NET

- Exit back to root

      exit

- Create password for tdoe

      ssh tdoe@localhost
      *Enter temp passowrd
      *Enter temp passowrd
      *Create new password
      *Confirm new password

- Verify a kerberos ticket was granted

      klist
      *Look for Default Principle tdoe@ECHO.NET

- Exit back to root

      exit

- Add an additional dns record for the webserver

      *This will be used for https://users.echo.net
      ipa dnsrecord-add echo.net users --a-rec 192.168.10.4


## Join `file01.echo.net` to the `ECHO.NET IPA Realm`
Do the following as root on `file01.echo.net`
- Follow the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)
    

## Configure `file01.echo.net` to use `ipa01.echo.net` as the NTP Server
- Follow the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-chronyd)


## Create Virtual Machines `router01.echo.net` and `smtp01.echo.net`

- Create the `router01.echo.net` virtual machine as specified in its VM-Specs file: [router01.echo.net](VM-Specs/router01.echo.net.md)
- Create the `smtp01.echo.net` virtual machine as specified in its VM-Specs file: [smtp01.echo.net](VM-Specs/smtp01.echo.net.md)


## Configure the Teaming connection on the Infrastructure NICs of `router01.echo.net`

- Create the team connection

      nmcli con add type team con-name team1 ifname team1 config '{"device": "team1","runner":{"name":"roundrobin"},"ports":{"eth2":{}, "eth3":{}}}'

- Assign a static IP Address to the team connection

      nmcli con mod team1 ipv4.addresses '192.168.11.2/24' ipv4.dns '192.168.11.3'
      nmcli con mod team1 ipv4.method manual

- Add network devicesto the team

      nmcli con add type ethernet con-name team1-port1 ifname eth2 master team1
      nmcli con add type ethernet con-name team1-port2 ifname eth3 master team1

- Ensure no other connections are using those devices

      nmcli con down eth2
      nmcli con down eth3

- Enable the slave connections

      nmcli con up team1-port1
      nmcli con up team1-port2

- Enable the team connection

      nmcli con up team1

- Verify connectivity

      ping 192.168.11.3
      teamdctl team1 state


## Configure the Teaming connection on the DMZ NICs of `router01.echo.net`

- Create the team connection

      nmcli con add type team con-name team0 ifname team0 config '{"device": "team0","runner":{"name":"roundrobin"},"ports":{"eth0":{}, "eth1":{}}}

- Assign a static IP Address to the team connection

      nmcli con mod team0 ipv4.addresses '192.168.10.2/24'
      nmcli con mod team0 ipv4.method manual

- Add network devicesto the team

      nmcli con add type ethernet con-name team0-port1 ifname eth0 master team0
      nmcli con add type ethernet con-name team0-port2 ifname eth1 master team0

- Ensure no other connections are using those devices

      nmcli con down eth0
      nmcli con down eth1

- Enable the slave connections

      nmcli con up team0-port1
      nmcli con up team0-port2

- Enable the team connection

      nmcli con up team0

- Verify connectivity

      ping 192.168.10.3
      teamdctl team0 state


## Enable `router01.echo.net` to route traffic between subnets
To enable `router01.echo.net` to route traffic between the subnets, do the following as root.

- Enable ip forwarding

      echo "1" > /proc/sys/net/ipv4/ip_forward

- Verify connection

      FROM smtp01.echo.net
      ping 192.168.11.3
      FROM ipa01.echo.net
      ping 192.168.10.3

- Make ip_forward change permanent

      vi /etc/sysctl.d/01-ip_forward.conf
      *Add the following line:
      net.ipv4.ip_forward = 1


## Disable firewalld on `router01.echo.net`
- Configuring firewalld to allow traffic between subnets is not very practical as most organizations will use a firewall appliance.

      systemctl disable firewalld --now


## Verify `router01.echo.net` configuration after reboot
- Reboot `router01.echo.net`

      reboot

- Verify firewalld is stopped/disabled

      systemctl status firewalld
      *Look for 'disabled;' and 'Active: inactive (dead)'

- Verify ipv4 ip_forwarding is enabled

      sysctl -a | grep ipv4.ip_forward
      *Look for 'net.ipv4.ip_forward = 1'


## Common Configuraton of `router01.echo.net` and `smtp01.echo.net`
- Add `Echo.Net Repo`
    - Add the `Echo.Net Repo` to `router01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)
    - Add the `Echo.Net Repo` to `smtp01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)

- Configure `ipa01.echo.net` as the NTP Server
    - Configure `ipa01.echo.net` as the NTP Server for `router01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-ntp)
    - Configure `ipa01.echo.net` as the NTP Server for `smtp01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-ntp)

- Join the `ECHO.NET IPA Realm`
    - Join `router01.echo.net` to the `ECHO.NET IPA Realm` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)
    - Join `smpt01.echo.net` to the `ECHO.NET IPA Realm` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)


## Configure `smtp01.echo.net` to recieve mail from `@ECHO.NET`
- Edit `main.cf` to allow mail reception

      vi /etc/postfix/main.cf
      *Change the directives as shown below:
      myorigin = $myhostname
      inet_interfaces = all
      #inet_interfaces = localhost
      #mydestination = $myhostname, localhost.$mydomain, localhost
      mydestination = $myhostname, licalhost.$mydomain, localhost, $mydomain
      **inet_protocolos = ipv4

- Allow smtp traffic through the firewall

      firewall-cmd --permanent --add-service smtp
      firewall-cmd --reload
      
- Restart postfix service

      systemctl restart postfix
      

## Configure `ipa01.echo.net` and `file01.echo.net` to relay email to `smtp01.echo.net`
- Configure relaying SMTP messages to `smtp01.echo.net` on `ipa01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-mail-relay)
- Configure relaying SMTP messages to `smtp01.echo.net` on `file01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-mail-relay)


## Create Virtual Machine `iscsi01.echo.net`
- Create the `iscsi01.echo.net` virtual machine as specified in its VM-Specs file: [iscsi01.echo.net](VM-Specs/iscsi01.echo.net.md)


## Perform common configuration steps on `iscsi01.echo.net`
- Add the `Echo.Net Repo` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)
- Configure `ipa01.echo.net` as the NTP Server by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-ntp)
- Join the `ECHO.NET IPA Realm` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)
- Configure relaying SMTP messages to `smtp01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-mail-relay)


## Configure iSCSI Target for NFS, SMB, and Database
- Login to `iscsi01.echo.net`
- Install targetcli

      yum install -y targetcli

- Enter targetcli prompt

      targetcli

- Create backstores

      cd /backstore/block
      create dev=/dev/vdb name=vdb-nfs
      create dev=/dev/vdc name=vdc-smb
      create dev=/dev/vdd name=vdd-db

- Create iSCSI Target

      cd /iscsi
      create wwn=iqn.2019-12.net.echo:vdb-nfs.vdc-smb
      create wwn=iqn.2019-12.net.echo:vdd-db
https://www.reddit.com/r/linuxadmin/comments/7iom6e/what_does_firewallcmd_addmasquerade_do/
      cd /iscsi/iqn.2019-12.net.echo:vdb-nfs.vdc-smb/tpg1/luns
      create storage_object=/backstores/block/vdb-nfs
      create storage_object=/backstores/block/vdc-smb
      cd /iscsi/iqn.2019-12.net.echo:vdd-db/tpg1/luns
      create storage_object=/backstores/block/vdd-db

- Create acls

      cd /iscsi/iqn.2019-12.net.echo:vdb-nfs.vdc-smb/tpg1/acl
      create wwn=iqn.2019-12.net.echo:file01
      cd /iscsi/iqn.2019-12.net.echo:vdd-db/tpg1/acl
      create wwn=iqn.2019-12.net.echo:db01

- Verify settings

      cd /
      ls

- Exit targetcli

      saveconfig
      exit

- Start and enable target.service

      systemctl enable target --now

- Allow iscsi through the firewall

      firewall-cmd --permanent --add-service iscsi-target
      firewall-cmd --reload


## Connect `file01.echo.net` to iSCSI Target `iscsi01.echo.net`
- Login to `file01.echo.net`
- Install iscsiadm command

      yum install -y iscsi-initiator-utils

- Backup existing initiator name

      mv /etc/iscsi/initiatorname.iscsi /tmp/

- Add initiator name from iSCSI Target ACL

      vi /etc/iscsi/initiatorname.iscsi
      *Add the following line:
      InitiatorName=iqn.2019-12.net.echo:file01

- Discover available iSCSI Targets

      iscsiadm --mode=discoverydb type=sendtargets --portal=iscsi01.echo.net --discover
      *Look for the two iSCSI targets created previously (vdb-nfs.vdc-smb & vdd-db)

- Login to vdb-nfs.vdc-smb target

      iscsiadm --mode node --targetname iqn.2019-12.net.echo:vdb-nfs.vdc-smb iscsi01.echo.net:3260 --login

- Verify connection

      lsscsi
      *Look for the two luns created previously

- Persistently mount iSCSI LUNs

      pvcreate /dev/sda /dev/sdb
      vgcreate vg_iscsi_nfs /dev/sda
      vgcreate vg_iscsi_smb /dev/sdb
      lvcreate -l 100%FREE -n lv_nfs /dev/vg_iscsi_nfs
      lvcreate -l 100%FREE -n lv_smb /dev/vg_iscsi_smb
      mkfs.ext4 /dev/vg_iscsi_nfs/lv_nfs
      mkfs.ext4 /dev/vg_iscsi_nfs/lv_smb
      mkdir -p /nfs /smb
      vim /etc/fstab
      *Add the followng line
      /dev/vg_iscsi_nfs/lv_nfs    /nfs    ext4    _netdev 0 0
      /dev/vg_iscsi_smb/lv_smb    /smb    ext4    _netdev 0 0
      mount -a

- Verify the iSCSI LUNs are mounted

      lsblk


## Configure NFS Shares
- Login to `file01.echo.net` as root
- Get admin kerberos ticket

      kinit admin

- Add nfs service

      ipa service-add nfs/file01.echo.net

- Download keytab from IPA Server

      ipa-getkeytab -s ipa01.echo.net -p nfs/file01.echo.net -k /etc/krb5.keytab

- Verify settings

      klist -k
      *Look for nfs/file01.echo.net@ECHO.NET

- Install nfs-utils

      yum install -y nfs-utils

- Start and enable nfs-server

      systemctl enable nfs-server --now

- Create directories to share

      mkdir -p /nfs/public /nfs/doegroup

- Ensure the correct permissions are on each directory

      chmod 777 /nfs/public
      chown -R root:jdoe /nfs/doegroup
      chmod 770 /nfs/doegroup

- Create NFS Shares

      vi /etc/exports
      *Add the following
      /nfs/public *(rw)
      /nfs/doegroup 192.168.11.0/24(rw,sec=krb5p)

- Change SELinux to appropriate context

      semanage fcontext -a -t public_content_rw_t "/nfs/public(/.*)?"
      semanage fcontext -a -t nfs_t "/nfs/doegroup(/.*)?"
      restorecon -Rv /nfs

- Reload nfs-server configuration

      exportfs -a
      *May need to fully reboot the server

- Allow NFS traffic through the firewall

      firewall-cmd --permanent --add-service nfs --add-service rpc-bind --add-service mountd
      firewall-cmd --reload

- Verify shares are exported

      showmount -e localhost
      *Look for both shares


## Mount NFS Shares on `ipa01.echo.net`
- Create a mount point for the NFS shares

      mkdir -p /mnt/nfs/doegroup /mnt/nfs/public

- Install nfs client packages

      yum install -y nfs4-acl-tools nfs-utils

- List shares on `file01.echo.net`

      showmount -e file01.echo.net

- Mount the NFS shares

      mount -t nfs file01.echo.net:/nfs/public /mnt/nfs/public
      mount -t nfs -o sec=krb5p file01.echo.net:/nfs/doegroup /mnt/nfs/doegroup

- Attempt to touch a file on the doegroup share (SHOULD FAIL)

      touch /mnt/nfs/doegroup/root.touch
      *Note this should fail because of kerberos authentication

- Switch to jdoe user

      ssh jdoe@localhost

- Touch a file to verify write access to the doegroup share

      touch /mnt/nfs/doegroup/jdoe.touch

- Add NFS shares to fstab

      vim /etc/fstab
      *Add the followng line
      file01.echo.net:/nfs/public      /mnt/nfs/public      nfs    _netdev 0 0
      file01.echo.net:/nfs/doegroup    /mnt/nfs/doegroup    nfs    _netdev 0 0

- Verify fstab enteries

      umount /mnt/nfs/public
      umount /mnt/nfs/doegroup
      mount -a
      mount | grep file01


## Configure SMB shares
- Login to `file01.echo.net`
- Install samba, samba-client, and cifs-utils

      yum install -y samba samba-client cifs-utils

- Create directories that will be shared

      mkdir -p /smb/public /smb/doegroup

- Change linux permissions

      chmod 777 /smb/public
      chown root:doegroup /smb/doegroup
      chmod 770 /smb/doegroup

- Change SELinux context

      semanage fcontext -a -t samba_share_t /smb/public
      semanage fcontext -a -t samba_share_t /smb/doegroup
      restorecon -Rv /smb

- Create samba users

      smbpasswd -a jdoe
      smbpasswd -a tdoe
      *Enter password as prompted

- Modify smb.conf using the example file

      vi /etc/samba/smb.conf
      *Add the following
      [public]
      comment = Public Share
      browseable = yes
      path = /smb/public
      guest ok = yes
      read only = no
      create mask = 777

      [doegroup]
      comment = Doegroup Share
      browseable = yes
      path = /smb/doegroup
      valid users = @doegroup
      write list = @doegroup

- Verify smb config

      testparm

- Start and Enable smb service

      systemctl enable smb --now

- Allow samba through the firewall

      firewall-cmd --permanent --add-service samba
      firewall-cmd --reload

- Verify shares are being advertised

      smbclient -L localhost


## Mount SMB Shares on `ipa01.echo.net`
- Login to `ipa01.echo.net`
- Install samba-client and cifs-utils

      yum install -y samba-client cifs-utils

- Create directories where the SMB shares will be mounted

      mkdir -p /mnt/smb/public /mnt/doegroup

- Create credentials file

      vim /root/jdoe-smb.creds
      *Add the following lines
      username=jdoe
      password=<PASSWORD>

- Get GID of doegroup

      ssh jdoe@localhost
      id
      *Look for '##############(jdoe)' <UID>  and '#############(doegroup)' <GID>
      exit

- Mount public share

      mount -t cifs -o credentials=/root/jdoe-smb.creds,file_mode=0666,dirmode=0777 //file01.echo.net/public /mnt/smb/public

- Test share permissions

      touch /mnt/smb/public/root.touch
      ssh jdoe@localhost
      touch /mnt/smb/public/jdoe.touch
      exit

- Mount doegroup share

      mount -t cifs -o credentials=/root/jdoe-smb.creds,uid=<UID>,gid=<GID>,file_mode=0660,dirmode=0770 //file01.echo.net/doegroup /mnt/smb/doegroup

- Test share permissions

      touch /mnt/smb/public/root.touch
      ssh jdoe@localhost
      touch /mnt/smb/public/jdoe.touch
      exit

- Add SMB shares to fstab

      vim /etc/fstab
      *Add the followng line
      //file01.echo.net/public      /mnt/smb/public      cifs    credentials=/root/jdoe-smb.creds,file_mode=0666,dirmode=0777,_netdev 0 0
      //file01.echo.net/doegroup    /mnt/smb/doegroup    cifs    credentials=/root/jdoe-smb.creds,uid=177200001,gid=177200003,file_mode=0660,dirmode=0770,_netdev 0 0

- Verify fstab enteries

      umount /mnt/smb/public
      umount /mnt/smb/doegroup
      mount -a
      mount | grep file01


## Create Virtual Machine `db01.echo.net`
- Create the `db01.echo.net` virtual machine as specified in its VM-Specs file: [db01.echo.net](VM-Specs/db01.echo.net.md)


## Perform common configuration steps on `db01.echo.net`
- Add the `Echo.Net Repo` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)
- Configure `ipa01.echo.net` as the NTP Server by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-ntp)
- Join the `ECHO.NET IPA Realm` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)
- Configure relaying SMTP messages to `smtp01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-mail-relay)


## Configure MariaDB Database (Non-Standard Port)
- Login to `db01.echo.net`
- Install MariaDB

      yum install -y mariadb-server

- Edit my.cnf to Non-Standard Port

      vim /etc/my.cnf
      *Add the following line
      port=3307

- Open firewall to allow sql queries on Non-Standard Port

      firewall-cmd --permanent --add-port=3307/tcp
      firewall-cmd --reload
- Add Non-Standard Port to SELinux mysqld ports

      yum install -y policycoreutils-python
      semanage port -a -t mysqld_port_t -p tcp 3307

- Verify port was added

      semanage port -l | grep mysqld
      *Look for 3307

- Start and Enable the mariadb service

      systemctl enable mariadb --now

- Secure mariadb

      mysql_secure_installation
      *Press Enter
      *y
      *Set a root password as prompted
      *y
      *y
      *y
      *y
      *y

- Connect to mariadb

      mysql -p

- Create a database

      CREATE DATABASE fancy_app;

- Verify database was created

      SHOW DATABASES;

- "CD" to the database

      USE fancy_app

- Create a USERS Table

      CREATE TABLE users (
          Id INT AUTO_INCREMENT,
          FirstName TEXT,
          LastName TEXT,
          PRIMARY KEY (Id)
      );

- Verify the table was created

      SHOW TABLES;
      DESCRIBE users;

- Insert two user records

      INSERT INTO users (
          FirstName, LastName
      ) VALUES (
          'Jenny', 'Jamison'
      );

      INSERT INTO users (
          FirstName, LastName
      ) VALUES (
          'Harry', 'Jangles'
      );

- Verify records were added

      SELECT * from users;

- Update Harry's name to Henry

      UPDATE users SET FirstName='Henry' WHERE FirstName='Harry';

- Verify row was updated

      SELECT * FROM users WHERE FirstName='Henry';

- Create a mariadb user for the web application to connect as

      CREATE USER appuser@'%' IDENTIFIED BY 'password';

- Verify the user was created

      SELECT user FROM mysql.user;

- Grant appuser full permissions to the fancy_app database

      GRANT ALL PRIVILEGES ON fancy_app.* TO appuser@'%';

- Verify the permissions were granted

      SHOW GRANTS FOR appuser;

- Exit mysql

      CTRL+c


## Create and Restore a MariaDB Backup
- Create back up fancy_app database

      mysqldump -u appuser -p fancy_app > /tmp/fancy_app-db.sql

- Change users table by adding an additional row

      mysql -u appuser -p
      *Enter password when prompted
      USE fancy_app;
      INSERT INTO users (FirstName, LastName) VALUES ('Paul','Simpson');

- Verify change

      SELECT * FROM users ORDER BY LastName;

- Exit mysql

      CTRL+c

- Restore fancy_app database

      mysql -u appuser -p fancy_app < /tmp/fancy_app-users.sql

- Verify change has been removed (backup restored)

      mysql -u appuser -p fancy_app
      USE fancy_app;
      SELECT * FROM users ORDER BY LastName;


## Create Virtual Machine `web01.echo.net`
- Create the `web01.echo.net` virtual machine as specified in its VM-Specs file: [web01.echo.net](VM-Specs/web01.echo.net.md)


## Perform common configuration steps on `web01.echo.net`
- Add the `Echo.Net Repo` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#add-the-echo.net-repo)
- Configure `ipa01.echo.net` as the NTP Server by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-ntp)
- Join the `ECHO.NET IPA Realm` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#join-ipa-realm)
- Configure relaying SMTP messages to `smtp01.echo.net` by following the steps in the [Common Configuration](Common_Configuration-Tasks.md#configure-mail-relay)


## Install and Configure Apache on `web01.echo.net`
- Login to `web01.echo.net`
- Install apache

      yum install -y httpd mod_ssl

- Start and Enable the service

      systemctl enable httpd --now

- Allow http/https traffic through the firewall

      firewall-cmd --permanent --add-service=http --add-service https
      firewall-cmd --reload


## Add HTTP and HTTPS VirtualHost for `users.echo.net`
- Login to `web01.echo.net`
- Create the directory that will be used for `users.echo.net`

      mkdir -p /web/users.echo.net

- Change SELinux context

      yum install -y policycoreutils-python
      semanage fcontext -a -t httpd_sys_content_t "/web/users.echo.net(/.*)?"
      restorecon -Rv /web

- Add the directory to httpd.conf

      vim /etc/httpd/conf/httpd.conf
      *Add the following lines:
      <Directory "/web/users.echo.net>
          AllowOverride None
          Require all granted
      </Directory>

- Add a VirtualHost configuration

      vim /etc/httpd/conf.d/users.echo.net.conf
      *Add the following lines
      <VirtualHost *:80>
      DocumentRoot /web/users.echo.net
      ServerName users.echo.net
      ErrorLog logs/users.echo.net-error_log
      ScriptAlias / /web/users.echo.net/cgi-bin/query_users.sh
      </VirtualHost>

- Copy VirtualHost configuration and modify for SSL

      cp /etc/httpd/conf.d/users.echo.net.conf /etc/httpd/conf.d/ssl_users.echo.net.conf
      vim /etc/httpd/conf.d/ssl_users.echo.net.conf
      *Change <VirtualHost *:80> to <VirtualHost *:443>


## Create 'Mock' application to query database
- Loging to `web01.echo.net`
- Install mysql client

      yum install -y mariadb

- Test the connection

      mysql -h db01.echo.net -P 3307 -u appuser -p
      *Enter password when prompted
      exit

- Test the query

      mysql -h db01.echo.net -P 3307 -u appuser -p<PASSWORD> --database=fancy_app -e "select * from users;"

- Create a cgi-bin directory under `users.echo.net` DocumentRoot

      mkdir -p /web/users.echo.net/cgi-bin

- Change the SELinux context of the cgi-bin directory

      semange fcontext -a -t httpd_sys_script_exec_t "/web/users.echo.net/cgi-bin(/.*)?"
      restorecon -Rv /web/users.echo.net/cgi-bin

- Enter the query into a script that will be ran when the webserver is accessed

      vi /web/users.echo.net/cgi-bin/query_users.sh
      *Add the following lines:
      #!/usr/bin/env bash
      echo -e "Content-type: text\n"
      echo "These are the users of fancy_app:"
      mysql -h db01.echo.net -P 3307 -u appuser -p<PASSWORD> --database=fancy_app -e "select * from users;"

      *Note:* The newline (\n) at the end of the header is required.

- Make the script executable

      chmod 755 /web/users.echo.net/query_users.sh

- Test the script

      /web/users.echo.net/query_users.sh
      *Should return a list of users

- Add 3307 to mysqld SELinux

      semange port -a -t mysqld_port_t -p tcp 3307

- Test the site

      curl users.echo.net
      *Should return script output
      curl -k https://users.echo.net

- Troubleshoot if needed by checking

      /var/logs/httpd/users.echo.net-error_log