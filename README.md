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
- during the first boot I've set up the time and a regular user for myself
- now that I've entered desktop first I’ll patch CentOS and install/configure the Firewall, Shorewall in this case, by entering the terminal and becoming root since I need to be root to enter the following commands.

```

su root
~~enter password~~
yum update
~~this will update all services and when prompted to, type in y as for yes~~

```
