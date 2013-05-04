---
layout: post
title: "Redundant Failover firewall with pf, pfsync and CARP on FreeBSD"
date: 2007-06-15 16:35
comments: false
categories: 
---

This is a step by step tutorial that should take most of a day.  I'm posting this here mostly as a reminder so that I can come back and read it when I need to build another firewall but hopefully it will be helpful to someone else also.

<!-- more -->

####Install FreeBSD

Download the disc 1 and disc 2 from [ftp://ftp.freebsd.org/pub/FreeBSD/releases/i386/ISO-IMAGES/6.2](ftp://ftp.freebsd.org/pub/FreeBSD/releases/i386/ISO-IMAGES/6.2)

Burn the iso images to cds and boot the target machine with disc 1.

Start a Standard installation.

Highlight any partitions and hit "d" to delete them, then hit "a" to use the entire disk, then hit "q" to continue.

Choose Standard for no boot manager.

Create partitions.  You can adjust the sizes of the partitions based on the size of your drive. The /usr/local and /usr/home partitions can go as low as 128MB since this won't be a common-user system and there won't be a lot of user-specific files or binaries...but the /usr partition should never go below 2,000MB since that's where all of your kernel source code and the ports tree is located. Here's a partition scheme if you have a 6GB drive:

    486MB swap partition (or at least 2x your RAM)
    512MB file system mounted as /
    512MB file system mounted as /tmp
    1267MB file system mounted as /var
    3115MB file system mounted as /usr
    128MB file system mounted as /usr/local
    128MB file system mounted as /usr/home

Press q to continue.

Highlight Kern-Developer and press space bar.

When asked about installing the ports collection choose yes.

Highlight exit and press enter.

Choose CD/DVD as the install media.

Last Chance, are you sure? Yes

When you see Congradulations, hit ok.

####FreeBSD Post-Install configuration

Would you like to configure any ethernet or SLIP/PPP network devices? Yes

Highlight your device that is connected to the internet and press enter.

Do you want to try IPv6? No

Do you want to try DHCP? Yes

Verify network info and update if necessary.

Do you want this machine to function as a gateway? Yes 

Do you want to configure inetd and the network services that it provides? No 

Would you like to enable SSH login? Yes 

Do you want to have anonymous FTP access to this machine? No 

Do you want to configure this machine as an NFS Server: No 

Do you want to configure this machine as an NFS Client: No 

Would you like to customize your system console settings? No

Would you like to set this machine's time zone now? Yes

Is your machine's CMOS clock is set to UTC? No

Select the appropriate time zone - by region, country, and then the applicable time zone. 

Would you like to enable Linux Binary compatibility? No

Does your system have a PS/2, serial, or bus mouse? Yes (unless, of course, it doesn't...) 

Would you like to browse the ports collection? Yes

####Install cvsup and bash

cvsup will be used to keep your system up to date.

Highlight net and press enter.

Highlight cvsup-without-gui and press space bar.

Tab to ok and press enter.

bash is a shell that we will use instead of the default sh

Highlight shells and press enter.

Highlight bash-2 and press space bar.

Tab to ok and press enter.

Tab to Install and press enter
Review your selections and press enter to install.

Would you like to add any user accounts? Yes (we need at least one so we can dis-allow root ssh access and require you to login as an unprivilaged user and su to root)

Highlight User and press enter.

Type the username, password, full name, and make the member group 0. (this is important so that your new user will be in the 'wheel' group and they will be able to su to root.  Also make the login shell /usr/local/bin/bash (I will create a user with the username admin so if you do the same you can follow the rest of this tutorial verbatim, otherwise just change admin to your username every time I use it later.

Set the root password.

Would you like to visit the general configuration menu for a chance to set any last options? Yes

Go to network then ntupdate and choose a server near you.

Highlight exit and press enter. (do this twice to get to the main menu)

Tab over to Exit Install and hit enter.

(System Reboots)

####Make "bash" the default shell for 'root' and perform an initial set up of root's bash environment. 

Log in as your non-privaleged user account. You should see a 'bash-2.05b$' prompt...indicating that bash was successfully installed. After you log in, then type 'su' to switch user to root. Enter the root password.

Type vipw and press enter.

Change the default shell from /bin/csh (at the end of the first line) to /usr/local/bin/bash.

If you like, you can change root's unoffical name from 'Charlie &' to 'Super-User' or whatever you like.  When you get mail from root, it will be marked as being from the name that you enter here.

Verify that your changes are successful. Press <Alt>-F2 to get another terminal, then log in as root. Verify that you're presented with the 'bash-2.05b#' prompt. If you are, then log out and go back to the 1st virtual terminal to continue working. If you don't see the bash prompt, then you need to go back to the previous step and figure out what you did wrong.

Create a file named .bashrc in /root that contains the following.

{% codeblock lang:bash %}
umask 077
PS1="[\u@\h \W]\\$ "
alias ls='ls -alFG'
{% endcodeblock %}

Also create a file named .bash_profile in /root that contains the following.

{% codeblock lang:bash %}
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin:/usr/local/sbin:$HOME/bin; export PATH
umask 077
PS1="[\u@\h \W]\\$ "
alias ls='ls -alFG'
{% endcodeblock %}

Change the permissions on both files.

    chmod 600 .bashrc
    chmod 600 .bash_profile

Test your settings again at the 2nd virtual terminal.  Log in as root, verify you're using the bash shell, your cursor line looks different (it has your userid and current working directory), and that you have colorized directory listings.  Close that session, return to the 1st virtual terminal, log out, and log back in and su to root.

Copy the files to your non privilaged user change the ownership of the new copies.

    cp /root/.bashrc /usr/home/admin/.bashrc
    cp /root/.bash_profile /usr/home/admin/.bash_profile
    chown admin /usr/home/admin/.bashrc
    chown admin /usr/home/admin/.bash_profile

####Redirect root's email to your email account.

    vi /etc/aliases

Uncomment # root: me@my.domain and change me@mydomain to your email address.

Save and exit.

Update the email alias database.

    newaliases

####Configure cvsup and update your source tree & ports collection.
After you configure cvsup and update your source and ports collection, you will want to re-run cvsup every once in a while to ensure your sources are up to date,  then recompile your kernel & system binaries to ensure you are using the latest versions with security patches applied.

    cp /usr/share/examples/cvsup/stable-supfile /etc
    chmod 600 /etc/stable-supfile
    vi /etc/stable-supfile

Type ":set num" in vi to see line numbers

Change line 68 to point to a CVS server near you. Here you'll find a list of CVSup servers [http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/cvsup.html#CVSUP-MIRRORS](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/cvsup.html#CVSUP-MIRRORS).  I ususlly go with cvsup2 or 3 because the main site reaches the maximum number of simultaneous users often.

Add these lines at the bottom of the file:

{% codeblock lang:bash %}
ports-net tag=CHANGE_THIS.FreeBSD.org
ports-shells tag=CHANGE_THIS.FreeBSD.org
{% endcodeblock %}

Syncronize your source tree with the CVS server...should take 30-60 minutes

    cvsup /etc/stable-supfile

Configure a custom kernel with support for ph, phsync, pflog, and carp

    cd /usr/src/sys/i386/conf
    cp GENERIC FIREWALL
    vi FIREWALL

In line 2 of the file (part of the main comment block) change the word, GENERIC, to your hostname, FIREWALL. 

On line 19 of the file (still part of the main comment block), change the word, GENERIC, to your hostname, FIREWALL 

On lines 22-24, comment out the "cpu" lines so that only the one for your specific chip is left.

On line 25, change the value of the ident parameter so that it's your hostname, FIREWALL 

Add the following lines to the bottom of the file.

{% codeblock lang:bash %}
# pf support
device pf
device pfsync
device pflog
device carp
{% endcodeblock %}

Recompile your kernel to the updated stable version.  (make buildworld will take about an hour and make buildkernel will take about 20 minutes)

    [root@firewall /]# cd /usr/src
    [root@firewall src]# echo "KERNCONF=FIREWALL" >> /etc/make.conf
    [root@firewall src]# make buildworld
    [root@firewall src]# make buildkernel
    [root@firewall src]# make installkernel

####Configure the SSH daemon and your user's DSA key files.
Modify the SSH daemon configuration file, /etc/ssh/sshd_config, so that it reads as follows.  I'm using port 8081 for ssh access instead of the usual port 21.  In the past I have had issues with hackers trying to use brute force tactics on my standard ports if they were open.

{% codeblock lang:bash %}
Port 8081
Protocol 2
#AddressFamily any
ListenAddress 192.168.1.1 # Put your internal interface's address here
#ListenAddress :: 

# HostKey for protocol version 1
#HostKey /etc/ssh/ssh_host_key
# HostKeys for protocol version 2
#HostKey /etc/ssh/ssh_host_dsa_key

# Lifetime and size of ephemeral version 1 server key
#KeyRegenerationInterval 1h
#ServerKeyBits 768

# Logging
# obsoletes QuietMode and FascistLogging
#SyslogFacility AUTH
LogLevel VERBOSE

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
StrictModes yes
#MaxAuthTries 6

RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
RhostsRSAAuthentication no
# similar for protocol version 2
HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# RhostsRSAAuthentication and HostbasedAuthentication
IgnoreUserKnownHosts yes
# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes

# Change to yes to enable built-in password authentication.
PasswordAuthentication no
PermitEmptyPasswords no

# Change to no to disable PAM authentication
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes

# Set this to 'no' to disable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication. Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM no

AllowTcpForwarding no
GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PrintMotd yes
PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
#UsePrivilegeSeparation yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS yes
#PidFile /var/run/sshd.pid
#MaxStartups 10
#PermitTunnel no

# no default banner path
#Banner /some/path

# override default of no subsystems
Subsystem sftp /usr/libexec/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
# ForceCommand cvs server
AllowUsers admin # Substitute your non-privilaged user for admin
{% endcodeblock %}

Generate an SSH key (version 2) for your user, by performing the following steps: 

    [root@firewall /root]# su - admin          *** substitute your non-privileged userid for 'admin'
    [admin@firewall admin]$ ssh-keygen -d   *** then accept the default DSA key name & enter a passphrase (twice)

Add the public copy of your user's version 2 key to their own authorized_keys file by typing the following steps: 

    [admin@firewall admin]$ cd .ssh
    [admin@firewall .ssh]$ cat id_dsa.pub > authorized_keys

Copy your user's private & public keys to other systems that you'll be using to SSH to your new firewall from. By default, the private & public key go into a user's '.ssh' directory on those systems. Without the private key on those remote systems, your firewall will not accept connections from them. If you're new to FreeBSD and need to know how to access the floppy drive, follow the following steps. 

    [root@firewall root]# mkdir /mnt/floppy                     # This will make an empty mount point to mount the floppy to
    [root@firewall root]# mount -t msdosfs /dev/fd0 /mnt/floppy # Insert a DOS-formatted floppy before you do this
    [root@firewall root]# cd /mnt/floppy
    [root@firewall floppy]# cp /home/admin/.ssh/id_dsa* .       # Copies all of your user's ssh key info to the floppy
    [root@firewall floppy]# ls                                  # List the contents of the floppy to verify the files are there
    [root@firewall floppy]# cd .. 
    [root@firewall mnt]# umount /mnt/floppy                     # Unmount the floppy

Now that you've copied your user's private & public keys to another system, remove them from your user's .ssh directory on the firewall. This is only a precaution so that it can't be stolen by a hacker and compromised.

Open up your /etc/hosts.allow file, delete all of the lines, and ensure that it reads as follows.  Note that 192.168.1.0 is the address space of your internal network in this example.  If you're using a different internal address space (e.g. 10.10.10.0), then make the appropriate modifications.

{% codeblock lang:bash %}
# hosts.allow access control file for "tcp wrapped" applications.
ALL : localhost 127.0.0.1 : allow
sshd : 192.168.1.0/255.255.255.0 : allow
ALL : ALL : deny
# If you want to allow a specific computer on the Internet to SSH into your
# system, replace the 'sshd' line above with one like this...but subsitute
# the X.X.X.X and subnet mask to suit your needs (e.g. one computer, entire subnet
# etc.). Also, make sure you allow inbound SSH from that same host/subnet
# in your /etc/ipf.rules file.
# sshd : 192.168.1.0/255.255.255.0 X.X.X.X/255.255.255.255 : allow
{% endcodeblock %}

####Install and configure AIDE (Advanced Intrusion Detection Environment) 
    [root@firewall /root]# cd /usr/ports/security/aide
    [root@firewall aide]# make install clean
    [root@firewall aide]# cp /usr/local/etc/aide.conf.sample /var/db/aide/aide.conf
    [root@firewall aide]# cd /var/db/aide
    [root@firewall aide]# aide --init
    [root@firewall aide]# mv databases/aide.db.new databases/aide.db

Create a cron job to check the integrity of your system every Sunday at 4AM: 

    [root@firewall /root]# cd /etc
    [root@firewall /etc]# vi crontab

Add the following line to the file:

{% codeblock lang:bash %}
0   4   *   *   7    root   /usr/local/bin/aide --check
{% endcodeblock %}

####Restrict crontab access/usage
Create the file /var/cron/allow and add the following lines to it. Be sure to substitute 'newuser' for whatever your non-privileged user account is.

{% codeblock lang:bash %}
root
newuser
{% endcodeblock %}

Edit /etc/crontab and comment out the 'at' job that runs every 5 minutes.

{% codeblock lang:bash %}
*/5 * * * * root /usr/libexec/atrun
{% endcodeblock %}

Chmod your /etc/crontab file so that it is only readable by root.

    [root@firewall /etc]# chmod 600 /etc/crontab

####Edit your /etc/rc.conf to look like this
This is my rc.conf file.  Your network interfaces will be named differently if you have different cards (e.g. xl0 and xl1 are 3com network cards in my machine and rl0 is an integrated NE2000 network card)

{% codeblock lang:bash %}
icmp_drop_redirects="YES"
ntpdate_enable="YES"
ntpdate_flags="north-america.pool.ntp.org"
sshd_enable="YES"
usbd_enable="YES"
syslogd_flags="-ss"
sshd_flags="-4"
pf_enable="YES"
pf_rules="/etc/pf.conf"
pf_flags=""
pflog_enable="YES"
pflog_logfile="/var/log/pflog"
pflog_flags=""
kern_securelevel_enable="YES"
kern_securelevel="2"
dhcpd_enable="YES"
gateway_enable="YES"
defaultrouter="208.180.xxx.xxx"
hostname="dell_firewall"
network_interfaces="xl0 xl1 rl0 lo0 pfsync0"
cloned_interfaces="carp0 carp1 carp2"

# Loopback Interface
ifconfig_lo0="inet 127.0.0.1"

# External Public Interface (for the secondary firewall use a different public ip.)
ifconfig_xl0="208.180.xxx.xxx"

# External Public Carp Interface
ifconfig_carp0="208.180.xxx.xxx vhid 1 pass foo"
ifconfig_carp0_alias0="208.180.xxx.xxx vhid 1 pass foo"
ifconfig_carp0_alias1="208.180.xxx.xxx vhid 1 pass foo"

# Internal Interface (for the secondary firewall change the ip address to 192.168.1.251)
ifconfig_xl1="192.168.1.250"

# Internal Carp Interface
ifconfig_carp1="192.168.1.1 vhid 2 pass foo"

# Heartbeat Interface (for the secondary firewall, change the ip address to 10.10.10.251)
ifconfig_rl0="10.10.10.250"

#Heartbeat Carp Interface
ifconfig_carp2="10.10.10.1 vhid 3 pass foo"

# PFSync Interface
ifconfig_pfsync0="up syncif rl0"
{% endcodeblock %}

####Add the following lines to the bottom of /etc/sysctl.conf
Type ifconfig with no parameters to view your network configuration.  Each real interface should have a line that says status: active when the ethernet cable is plugged in.  If any of them do not, then preempt will not work.  Note that the changes we made to rc.conf do not show up because we have not rebooted yet.

{% codeblock lang:bash %}
net.inet.tcp.blackhole=2
net.inet.udp.blackhole=1

# if one interface fails then all will fail over
net.inet.carp.preempt=1

net.inet.tcp.sendspace=65536
net.inet.tcp.recvspace=65536
{% endcodeblock %}

####Configure the main firewall /etc/pf.conf

{% codeblock lang:bash %}
################################################################################
# Macro and lists
################################################################################
lop_if  ="lo0"
pfs_if  ="dc0"
ext_if  ="xl0"
int_if  ="xl1"
ext_carp ="carp0"

dns_srv  = "{ 208.180.xxx.xxx, 208.180.xxx.xxx }"
int_fw   = "{ 208.180.xxx.xxx 192.168.1.1 }"
chat_srv = "{ 192.168.1.119 }"

# Allowed incoming ICMP types
icmp_types = "{ echorep, echoreq, timex, paramprob, unreach code needfrag }"

# Private networks (RFC 1918)
priv_nets = "{ 127.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8 }"

mail_ports         = "{ 25, 110 }"
web_ports          = "{ www, https }"
ftp_ports          = "{ 8084 9000 9001 }"
firewall_ssh_ports = "{ 8081 }"
web_ssh_ports      = "{ 8082 }"
chat_ports         = "{ 8080 }"
dns_ports          = "{ 53 }"

################################################################################
# Options, scrub and NAT
################################################################################
set block-policy drop
set skip on $lop_if

scrub in

# NAT outgoing connections
nat on $ext_if from $int_if:network to any -> $ext_if

################################################################################
# Redirection
################################################################################
rdr on $ext_if proto tcp from any to $ext_carp port $chat_ports -> $chat_srv
rdr on $ext_if proto tcp from any to $ext_carp port $ftp_ports -> 192.168.1.80
rdr on $ext_if proto tcp from any to $ext_carp port $mail_ports -> 192.168.1.80
rdr on $ext_if proto tcp from any to $ext_carp port $web_ssh_ports -> 192.168.1.80
rdr on $ext_if proto tcp from any to 208.180.xxx.xxx port $web_ports -> 192.168.1.80
rdr on $ext_if proto tcp from any to 208.180.xxx.xxx port $web_ports -> 192.168.1.81
rdr on $ext_if proto tcp from any to 208.180.xxx.xxx port $web_ports -> 192.168.1.82

################################################################################
# Filtering Rules
################################################################################
block log all
antispoof quick for $int_if

pass in quick on $ext_if inet proto tcp from 208.180.xxx.xxx to $int_fw port $firewall_ssh_ports keep state

pass in quick on $int_if inet proto udp from any to any port 123 keep state
pass out quick on $ext_if inet proto udp from any to any port 123 keep state

pass quick on $pfs_if proto pfsync
pass quick proto carp

block in quick on $ext_if from $priv_nets to any
block out quick on $ext_if from any to $priv_nets

pass in quick on $ext_if inet proto tcp from any to $int_if:network port $mail_ports \
     synproxy state
pass out quick on $int_if inet proto tcp from any to $int_if:network port $mail_ports \
     keep state
pass in quick on $int_if inet proto tcp from $int_if:network port $mail_ports to any \
     keep state
pass out quick on $ext_if inet proto tcp from $int_if:network port $mail_ports to any \
     modulate state

pass in  quick on $ext_if inet proto tcp from any to $int_if:network port $web_ports \
     synproxy state
pass out quick on $int_if inet proto tcp from any to $int_if:network port $web_ports \
     keep state

pass in quick on $ext_if inet proto tcp from any to $chat_srv port $chat_ports \
     synproxy state
pass out quick on $int_if inet proto tcp from any to $chat_srv port $chat_ports \
     keep state

pass in  quick inet proto icmp all icmp-type $icmp_types
pass out quick inet proto icmp all

pass in quick on $int_if inet proto { tcp, udp } from $int_if:network to $dns_srv port domain keep state

pass out quick on $ext_if inet proto { tcp, udp } from { $ext_if $int_if:network} to $dns_srv port domain keep state

pass in  quick on $int_if inet proto tcp from $int_if:network to any port $web_ports
pass out quick on $ext_if inet proto tcp from { $ext_if $int_if:network } to any port $web_ports \
     modulate state
{% endcodeblock %}

####Configure the system to mount partitions restrictive
Modify the /etc/fstab file with vi so that we can change how each partition is mounted...to ensure that hackers can do at little as possible if they (by chance alone) hack the box. Essentially, we're restricting some of the partitions so that they are 'nosuid', 'noexec', and 'ro'. The original /etc/fstab should look something like this. Yours might look a little different...the first column (device names) might be a little different, but that's OK. The stuff we'll be modifying is in the 4th column. 

{% codeblock lang:bash %}
# Device    Mountpoint  FStype  Options   Dump  Pass#
/dev/ad0s1b none        swap    sw        0     0
/dev/ad0s1a /           ufs     rw        1     1
/dev/ad0s1d /tmp        ufs     rw        2     2
/dev/ad0s1f /usr        ufs     rw        2     2
/dev/ad0s1h /usr/home   ufs     rw        2     2
/dev/ad0s1g /usr/local  ufs     rw        2     2
/dev/ad0s1e /var        ufs     rw        2     2
/dev/acd0   /cdrom      cd9660  ro,noauto 0     0
{% endcodeblock %}

First, copy the original /etc/fstab file to /etc/fstab.original<br>
Then, make another copy of the /etc/fstab file and call it /etc/fstab.restrictive<br>
Then, modify the /etc/fstab.restrictive file so that it reads as follows: 

{% codeblock lang:bash %}
# Device    Mountpoint  FStype  Options                 Dump  Pass#
/dev/ad0s1b none        swap    sw                      0     0
/dev/ad0s1a /           ufs     rw,nosuid               1     1
/dev/ad0s1d /tmp        ufs     rw,noexec,nosuid,nodev  2     2
/dev/ad0s1f /usr        ufs     ro                      2     2
/dev/ad0s1h /usr/home   ufs     rw,noexec,nosuid        2     2
/dev/ad0s1g /usr/local  ufs     ro,nosuid               2     2
/dev/ad0s1e /var        ufs     rw,noexec,nosuid        2     2
/dev/acd0   /cdrom      cd9660  ro,noauto               0     0
{% endcodeblock %}

Next, copy your new /etc/fstab.restrictive file and over-write the original /etc/fstab...so that your "real" fstab file has the restrictive settings, and you have the two other config files available (the original and restrictive one). 

    [root@firewall etc]# cp /etc/fstab.restrictive /etc/fstab

This will make adding new software, etc. much more difficult since /usr and /usr/local are mounted read-only. This means that programs which try to install their user-land programs in /usr/local/bin will fail during their install programs. And cvsup...which will try to update the kernel's source code in /usr/src and the ports in /usr/ports...well, they're now read-only because they fall under /usr. So, mounting your partitions in a very restrictive way will limit what the hacker can do on your system, but it makes software installs and kernel upgrades more difficult (or impossible...if the partitions are still mounted in a restrictive way).

Given that, if you want to add new software or upgrade the kernel & ports tree source code, you'll need to 

1. Change the partition's mounting in /etc/fstab back to their original values by copying your /etc/fstab.original file to /etc/fstab. 
2. Bump the kernel security level back down to "1" by setting the kern_securelevel paramater in your /etc/rc.conf file, and then 
3. Reboot the machine (this will not cause downtime because the secondary firewall will take over while this machine is rebooting) 
4. Update your sources with cvsup, then make buildworld, make buildkernel, and make installworld

Then when you're done upgrading, recompiling, and installing, do the steps in reverse: 

1. Change the partition's mounting in /etc/fstab to their restrictive values by copying your /etc/fstab.restrictive file to /etc/fstab. 
2. Bump the kernel security level back up to "2" by setting the kern_securelevel paramater in your /etc/rc.conf file, and then 
3. Reboot the machine 

This is the price you pay for a VERY, VERY secure machine. Reboot the machine so we can finish the job... 

    [root@firewall /etc]# shutdown -r now

If the system doesn't reboot, it means that you probably made an error in the kernel configuration file...possibly setting the wrong type of CPU. DON'T PANIC. We can still boot the machine so that you can fix the error. To boot into the original version of the kernel, following the steps, below: 

Reboot the machine (power off, then on) 
When you reboot the machine and get to the bootloader screen, select option 6, "Escape to loader prompt". This will give you an "OK" prompt at the bottom of the screen. Boot from your "old" kernel.

    OK unload
    OK boot /boot/kernel.old/kernel

After the old kernel boots, you'll want to copy the "old" kernel to a safe place before you recompile a new kernel. This is an important thing to do since "kernel.old" is overwritten when you install a new kernel. To do this, type the following commands: 

    [root@firewall /]# cp -R /boot/kernel.old /boot/kernel.good

If subsequent kernel compiles still don't work, you can always manually boot off your good kernel from the "OK" prompt until you resolve the problem...just substitute "kernel.good" for "kernel.old" in the commands listed above. 

Now that you've saved a copy of your "good" kernel, modify your kernel configuration file and fix whatever was causing the problems, recompile & install, and then reboot and continue with the next step. 

After the system comes back up, you'll want to re-generate the AIDE database and replace the old one. Since you updated the kernal and all of the system binaries, the AIDE database signatures of those files is out of date. If you don't update the AIDE database, AIDE will find thousands of "changes" to the system binaries when it runs for the first time at 4AM in the morning. To update the database so that it has signatures for the newest kernel & system binaries, etc, just type the following commands: 

{% codeblock lang:bash %}
[root@firewall /]# cd /var/db/aide
[root@firewall aide]# aide --init
[root@firewall aide]# mv databases/aide.db.new databases/aide.db
{% endcodeblock %}

After you do this you should have a completely working firewall.

####Resources
[Failover Firewall HowTo](http://www.countersiege.com/doc/pfsync-carp/#configuration)  
[How to Build a FreeBSD-STABLE Firewall with IPFILTER](http://www.schlacter.net/public/FreeBSD-STABLE_and_IPFILTER.html)  
[Redundant Failover Firewall with pf, pfsync and CARP under FreeBSD.](http://www.familywilson.ca/pf-carp-freebsd-redundant-firewall/)  