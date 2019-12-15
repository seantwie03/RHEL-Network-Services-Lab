# Network Services Lab - Common Configuration
This document lists all the configuration tasks that must be performed multiple times.  The main configuration document will reference this, rather than repeating the same steps over and over.

## Add the `Echo.Net Repo`

- Remove default CentOS7 repos

      mv /etc/yum.repos.d/*.repo /tmp/

- Congifure `Echo.Net Repo`

      vi /etc/yum.repos.d/echo.net.repo
      *Add the following
      [Echo.Net.Repo]
      name=Echo.Net Repo
      baseurl=http://192.168.11.5/repo
      enabled=1
      gpgcheck=1
      gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

- Test repo configuration

      yum install -y vim       


## Configure NTP

- Install chrony if it is not already installed:

      yum install -y chrony

- Ensure NTP is enabled

      timedatectl
      *Look for "NTP Enabled: yes"

- If NTP is not enabled, enable it

      timedatectl set-ntp 1

- Add `ipa01.echo.net` as NTP Server and `smtp01.echo.net` as NTP peer

      vi /etc/chrony.conf
      *Change the file as shown below:
      #server 0.centos.pool.ntp.org iburst
      #server 1.centos.pool.ntp.org iburst
      #server 2.centos.pool.ntp.org iburst
      #server 3.centos.pool.ntp.org iburst
      server ipa01.echo.net iburst
      peer smtp01.echo.net iburst

- Restart the chronyd service

      systemctl restart chronyd

- Wait 5 to 10 seconds
- Verify the configuration

      timedatectl
      *Look for NTP enabled: yes and NTP synchronized yes

- Verify chonyd operation

      chronyc sources -v

- Check time

      date


## Join IPA Realm

- Install ipa-client

      yum install -y ipa-client

- Ensure time is synchronized with `ipa01.echo.net`

      timedatectl
      *Look for 'NTP synchronized: yes'

- Join LDAP realm

      ipa-client-install --enable-dns-updates
      When asked to configure the system answer: yes
      When asked which user to enroll as type 'admin' and enter the admin password

- Verify kerberos

      kinit admin

- Verify DNS

      dig $(hostname)
      *Look for 'status: NOERROR'
      *Ensure ';; ANSWER SECTION' is displaying the correct hostname and IP Address

- Verify PTR record synchronization

      dig -x <IP_ADDRESS>
      *Note: <IP_ADDRESS> of server that just joined the IPA Realm
      *Look for 'status: NOERROR'


## Configure Mail Relay
To configure `smtp01.echo.net` as the mail relay, do the following as root:

- Edit the `main.cf` file

      vim /etc/postfix/main.cf
      *Modify the directives as shown below:
      relayhost = [smtp01.echo.net]
      *Note: the square brackets turn off mx lookups. 

- Restart postfix to apply the changes

      systemctl restart postfix

- Install mailx to send emails

      yum install -y mailx

- Send a test email

      mailx -s "Test from $(hostname)" root@smtp01.echo.net < .

- Verify email was sent

      postqueue -p

- Verify email was received

      *On `smtp01.echo.net`
      cat /var/spool/mail/root
      *Look for 'Subject: Test from <HOSTNAME>.echo.net'