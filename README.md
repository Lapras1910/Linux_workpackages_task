Linux_workpackages_task
===================

Task at hand,  I'd like to start by setting up my own CentOS 6.5 instance on a virtual machine.
Since I never used that ''variation'' of Linux before. I'll be using 86x-64x version of CentOs 6.5 since I got my hands on that version first.

Installationa_and_setting_up
=====================

- First dialogue offers multiple options, I've selected '' Install or upgrade an existing system''
- After testing and verification of the CentOS image the installation proceeds, with the initial setup (selecting installation language, selecting keyboard for the system etc. )
- I've given it the hostname ''centhost''
- I've set my root password to '' P4$$word''
- next dialogue select '' Use All Space'' option
- next pick desktop install 
- after the installation is finished I'm prompted to reboot 
- during the first boot I've set up the time and a regular user for myself ''Ivan'' in this case
- now that I've entered desktop first i'll disable root and ad my user to sudoers
```

su root
~~enter password~~
yum install nano -y
usermod -g wheel Ivan
yum install sudo -y

```
-nano is a command line text editor which I'll need to edit the following files
-also I added my user to the wheel group as administrator
-and installed sudo
- now for some editing
```

nano /etc/sudoers
~~now find end set this lines to...~~
## Same thing without a password
%wheel  ALL=(ALL)     NOPASSWD: ALL
~~save changes~~
nano /etc/ssh/sshd_config
~~find and edit~~
PermitRootLogin no
~~save changes~~
/etc/init.d/sshd restart

```
-now I added my user to sudoers and disabled Root Logins
-now I’ll install the IT automation tool Ansible
- but first I have to install the EPEL repository 


```

wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm -Uvh epel-release-6*.rpm

```
- the EPEL repository provides useful software packages that are not included in the official CentOS repositories.
```
yum install ansible

```
- we'll return back to ansible later on
- now i will setup SHH client fail2ban
```

yum install fail2ban -y

```
-now I'll configure the basic setup
```

nano /etc/fail2ban/jail.conf
~~edit this lines to~~
# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space separator.
Ignoreip = 127.0.0.1 10.0.2.15/24

# "bantime" is the number of seconds that a host is banned.
Bantime = 3600

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
Findtime = 600

# "maxretry" is the number of failures before a host get banned.
maxretry = 3

```
- 10.0.2.15/24 is my IPv4 adress so i added it the exceptions and I also extended the ban time to 3600 seconds
```

[ssh-iptables]

enabled  = true
filter   = sshd
action   = iptables[name=SSH, port=ssh, protocol=tcp]
           sendmail-whois[name=SSH, dest=root, sender=fail2ban@example.com]
logpath  = /var/log/secure
maxretry = 5
~~save changes~~
```

- now to setup public key authentication
```
[root@centhost ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
d4:a6:4f:31:49:c9:87:b8:ef:55:e7:2b:ab:31:20:1c root@centhost
The key's randomart image is:
+--[ RSA 2048]----+
|         o.o     |
|        .o+..    |
|       E..*.     |
|      ..oo o  . .|
|       oSo.  . o |
|        .oo .   .|
|         ..+    .|
|          . o. . |
|           ...o  |
+-----------------+ 
~~image got slightly messed up during c-p~~
```
- now I'll tighten up file system permission
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
```
- now I'll copy the public key to the machine I want to SSH and fix permissions

```
ssh root@cnode01 'mkdir -p /root/ssh'
scp /root/.ssh/id_rsa.pub root@cnode01:/root/.ssh/authorized_keys
ssh root@cnode01 'chmod 700 /root/.ssh'
ssh root@cnode01 'chmod 600 /root/.ssh/*'

```
