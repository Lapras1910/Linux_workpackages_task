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
- now I only need to install scp on the remote machine and I'm good to go
```
ssh root@cnode01 'yum install openssh-clients'
ssh cnode01

```
- next I'll install and configure shorewall
```
yum install shorewall -y

```
- now i'll configure the zones
```
nano /etc/shorewall/zones
~~update to~~
fw	firewall
wan	ipv4
lan	ipv4
~~save changes~~
```
- now for the interfaces
```
nano /etc/shorewall/interfaces
~~update to~~
wan	eth1	-	routefilter,blacklis,tcpflags,logmartians,nosmurfs
lan	eth0
~~save changes~~
```
- now policy
```
nano /etc/shorewall/policy
~~update to~~
## allow lan to all and firewall to all (outgoing to internet) but no traffic from wan/internet to lan or firewall itself
lan     all     ACCEPT
$FW     all     ACCEPT
wan     all     DROP    info
# this must be last rule
all     all     REJECT  info
~~save changes~~
```
- now that I have set the policy to drop all wan connections,  and I need some exceptions to make use of SSH
```
nano /etc/shorewall/rules
~~update to~~
 ACTION SOURCE	DEST	PROTO	DEST	
						PORT(s)
ACCEPT	wan	$FW	tcp		22
~~save changes~~
```
- now that SSH goes trough shorewall I'm free to adjust the rules later on should the situation demand more exceptions
- now for the black list, shorewall checks incoming packages against the /etc/shorewall/blacklist filewhich is used to perform static blacklisting by source adress( IP or MAC), or by applicationŽ
```
nano /etc/shorewall/blacklist
~~example blacklist entry~~
173.252.120.6
199.16.156.230
~~save changes~~
```
- now we need to enable shorewall on startup and check for errors
```
nano /etc/shorewall/shorewall.conf
~~update to~~
STARTUP_ENABLED=Yes
~~save changes~~
shorewall check
```
- if no errors come up I only have to save existing firewall rules 
```

iptables-save > /root/old.firewall.config
```



WORKPACKAGE 1: Mail Server
========================



WORKPACKAGE 2: Application Server
=============================
-I'll start by getting all the needed software, since CentOS comes with Python 2.6 I'll get Python 3.4 and 2.7 first
- In order to compile Python I must first install the development tools 
```
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```
-now the installation of Phyton 3.4
```
wget http://python.org/ftp/python/3.4.0/Python-3.4.0.tar.xz
tar xf Python-3.4.0.tar.xz
cd Python-3.4.0
./configure --prefix=/usr/local --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make
make altinstall
cd ..
```
-now the installation off Python 2.7
```
wget http://python.org/ftp/python/2.7.0/Python-2.7.0.tar.xz
tar xf Python-2.7.0.tar.xz
cd Python-2.7.0
./configure --prefix=/usr/local --enable-unicode=ucs4 --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make
make altinstall
cd ..
```
- now for virtualenv for both versions of Python
```
mkdir ~/src
cd ~/src
wget http://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.5.2.tar.gz#md5=fbcefbd8520bb64bc24a560c6019a73c
tar -zxvf virtualenv-1.5.2.tar.gz
cd virtualenv-1.5.2/
~/.localpython/bin/python setup.py install
virtualenv ve -p /home/Ivan/.localpython/bin/python2.7
source ve/bin/activate

tar -zxvf virtualenv-1.5.2.tar.gz
cd virtualenv-1.5.2/
~/.localpython/bin/python setup.py install
virtualenv ve -p /home/Ivan/.localpython/bin/python3.4
source ve/bin/activate
```

-now for virtualenvwrapper
```
pip install virtualenvwrapper
```
