---
layout: post
title:  "Setup Apple Time Machine network drive with Samba on Ubuntu 22.04"
date:   2025-10-03 12:25:14 +0800
tags:   dev--network os--ubuntu
---

The backup drive for my Mac's Time Machine fails this morning.
Therefore, I spent several hours working on setting up an Ubuntu 22.04 home server to serve as a network backup drive.

## Install Samba

Reference: <https://ubuntu.com/tutorials/install-and-configure-samba>.

Install samba with:

```bash
sudo apt update
sudo apt install samba
```

Then, set up a samba user account using the following command:

```bash
sudo smbpasswd -a username
```

The command will prompt for a password for the new account.
Note that the username must belong to a system account.
In my case, I use the same username and password as the home server's login user and password.

Samba comes with the following default config `/etc/samba/smb.conf`, which can be edited with `sudo vim /etc/samba/smb.conf`:

```conf
#
# Sample configuration file for the Samba suite for Debian GNU/Linux.
#
#
# This is the main Samba configuration file. You should read the
# smb.conf(5) manual page in order to understand the options listed
# here. Samba has a huge number of configurable options most of which
# are not shown in this example
#
# Some options that are often worth tuning have been included as
# commented-out examples in this file.
#  - When such options are commented with ";", the proposed setting
#    differs from the default Samba behaviour
#  - When commented with "#", the proposed setting is the default
#    behaviour of Samba but the option is considered important
#    enough to be mentioned here
#
# NOTE: Whenever you modify this file you should run the command
# "testparm" to check that you have not made any basic syntactic
# errors.

#======================= Global Settings =======================

[global]

## Browsing/Identification ###

# Change this to the workgroup/NT-domain name your Samba server will part of
   workgroup = WORKGROUP

# server string is the equivalent of the NT Description field
   server string = %h server (Samba, Ubuntu)

#### Networking ####

# The specific set of interfaces / networks to bind to
# This can be either the interface name or an IP address/netmask;
# interface names are normally preferred
;   interfaces = 127.0.0.0/8 eth0

# Only bind to the named interfaces and/or networks; you must use the
# 'interfaces' option above to use this.
# It is recommended that you enable this feature if your Samba machine is
# not protected by a firewall or is a firewall itself.  However, this
# option cannot handle dynamic or non-broadcast interfaces correctly.
;   bind interfaces only = yes



#### Debugging/Accounting ####

# This tells Samba to use a separate log file for each machine
# that connects
   log file = /var/log/samba/log.%m

# Cap the size of the individual log files (in KiB).
   max log size = 1000

# We want Samba to only log to /var/log/samba/log.{smbd,nmbd}.
# Append syslog@1 if you want important messages to be sent to syslog too.
   logging = file

# Do something sensible when Samba crashes: mail the admin a backtrace
   panic action = /usr/share/samba/panic-action %d


####### Authentication #######

# Server role. Defines in which mode Samba will operate. Possible
# values are "standalone server", "member server", "classic primary
# domain controller", "classic backup domain controller", "active
# directory domain controller".
#
# Most people will want "standalone server" or "member server".
# Running as "active directory domain controller" will require first
# running "samba-tool domain provision" to wipe databases and create a
# new domain.
   server role = standalone server

   obey pam restrictions = yes

# This boolean parameter controls whether Samba attempts to sync the Unix
# password with the SMB password when the encrypted SMB password in the
# passdb is changed.
   unix password sync = yes

# For Unix password sync to work on a Debian GNU/Linux system, the following
# parameters must be set (thanks to Ian Kahan <<kahan@informatik.tu-muenchen.de> for
# sending the correct chat script for the passwd program in Debian Sarge).
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

# This boolean controls whether PAM will be used for password changes
# when requested by an SMB client instead of the program listed in
# 'passwd program'. The default is 'no'.
   pam password change = yes

# This option controls how unsuccessful authentication attempts are mapped
# to anonymous connections
   map to guest = bad user


########## Domains ###########

#
# The following settings only takes effect if 'server role = classic
# primary domain controller', 'server role = classic backup domain controller'
# or 'domain logons' is set
#

# It specifies the location of the user's
# profile directory from the client point of view) The following
# required a [profiles] share to be setup on the samba server (see
# below)
;   logon path = \\%N\profiles\%U
# Another common choice is storing the profile in the user's home directory
# (this is Samba's default)
#   logon path = \\%N\%U\profile

# The following setting only takes effect if 'domain logons' is set
# It specifies the location of a user's home directory (from the client
# point of view)
;   logon drive = H:
#   logon home = \\%N\%U

# The following setting only takes effect if 'domain logons' is set
# It specifies the script to run during logon. The script must be stored
# in the [netlogon] share
# NOTE: Must be store in 'DOS' file format convention
;   logon script = logon.cmd

# This allows Unix users to be created on the domain controller via the SAMR
# RPC pipe.  The example command creates a user account with a disabled Unix
# password; please adapt to your needs
; add user script = /usr/sbin/adduser --quiet --disabled-password --gecos "" %u

# This allows machine accounts to be created on the domain controller via the
# SAMR RPC pipe.
# The following assumes a "machines" group exists on the system
; add machine script  = /usr/sbin/useradd -g machines -c "%u machine account" -d /var/lib/samba -s /bin/false %u

# This allows Unix groups to be created on the domain controller via the SAMR
# RPC pipe.
; add group script = /usr/sbin/addgroup --force-badname %g

############ Misc ############

# Using the following line enables you to customise your configuration
# on a per machine basis. The %m gets replaced with the netbios name
# of the machine that is connecting
;   include = /home/samba/etc/smb.conf.%m

# Some defaults for winbind (make sure you're not using the ranges
# for something else.)
;   idmap config * :              backend = tdb
;   idmap config * :              range   = 3000-7999
;   idmap config YOURDOMAINHERE : backend = tdb
;   idmap config YOURDOMAINHERE : range   = 100000-999999
;   template shell = /bin/bash

# Setup usershare options to enable non-root users to share folders
# with the net usershare command.

# Maximum number of usershare. 0 means that usershare is disabled.
#   usershare max shares = 100

# Allow users who've been granted usershare privileges to create
# public shares, not just authenticated ones
   usershare allow guests = yes

#======================= Share Definitions =======================

# Un-comment the following (and tweak the other settings below to suit)
# to enable the default home directory shares. This will share each
# user's home directory as \\server\username
;[homes]
;   comment = Home Directories
;   browseable = no

# By default, the home directories are exported read-only. Change the
# next parameter to 'no' if you want to be able to write to them.
;   read only = yes

# File creation mask is set to 0700 for security reasons. If you want to
# create files with group=rw permissions, set next parameter to 0775.
;   create mask = 0700

# Directory creation mask is set to 0700 for security reasons. If you want to
# create dirs. with group=rw permissions, set next parameter to 0775.
;   directory mask = 0700

# By default, \\server\username shares can be connected to by anyone
# with access to the samba server.
# Un-comment the following parameter to make sure that only "username"
# can connect to \\server\username
# This might need tweaking when using external authentication schemes
;   valid users = %S

# Un-comment the following and create the netlogon directory for Domain Logons
# (you need to configure Samba to act as a domain controller too.)
;[netlogon]
;   comment = Network Logon Service
;   path = /home/samba/netlogon
;   guest ok = yes
;   read only = yes

# Un-comment the following and create the profiles directory to store
# users profiles (see the "logon path" option above)
# (you need to configure Samba to act as a domain controller too.)
# The path below should be writable by all users so that their
# profile directory may be created the first time they log on
;[profiles]
;   comment = Users profiles
;   path = /home/samba/profiles
;   guest ok = no
;   browseable = no
;   create mask = 0600
;   directory mask = 0700

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

# Windows clients look for this share name as a source of downloadable
# printer drivers
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
# Uncomment to allow remote administration of Windows print drivers.
# You may need to replace 'lpadmin' with the name of the group your
# admin users are members of.
# Please note that you also need to set appropriate Unix permissions
# to the drivers directory for these users to have write rights in it
;   write list = root, @lpadmin
```

I delete the printer sections:

```diff
diff --git 1/smb.conf 2/smb.conf
index 8fe8be8..55c516a 100644
--- 1/smb.conf
+++ 2/smb.conf
@@ -217,23 +217,23 @@
 ;   create mask = 0600
 ;   directory mask = 0700
 
-[printers]
-   comment = All Printers
-   browseable = no
-   path = /var/spool/samba
-   printable = yes
-   guest ok = no
-   read only = yes
-   create mask = 0700
+#[printers]
+#   comment = All Printers
+#   browseable = no
+#   path = /var/spool/samba
+#   printable = yes
+#   guest ok = no
+#   read only = yes
+#   create mask = 0700
 
 # Windows clients look for this share name as a source of downloadable
 # printer drivers
-[print$]
-   comment = Printer Drivers
-   path = /var/lib/samba/printers
-   browseable = yes
-   read only = yes
-   guest ok = no
+#[print$]
+#   comment = Printer Drivers
+#   path = /var/lib/samba/printers
+#   browseable = yes
+#   read only = yes
+#   guest ok = no
 # Uncomment to allow remote administration of Windows print drivers.
 # You may need to replace 'lpadmin' with the name of the group your
 # admin users are members of.
```

## Setup Samba for Time Machine

Reference: <https://blog.jhnr.ch/2023/01/09/setup-apple-time-machine-network-drive-with-samba-on-ubuntu-22.04/>.

To make Samba work we need `samba-vfs-modules` package.
Install it with:

```bash
sudo apt install samba-vfs-modules
```

Create the backup directory in the home server:

```bash
mkdir /data/MyMacTimeMachine
```

We further configure `/etc/samba/smb.conf` as what follows:

```diff
diff --git 2/smb.conf 3/smb.conf
index 55c516a..98b10c5 100644
--- 2/smb.conf
+++ 3/smb.conf
@@ -99,6 +99,21 @@
 # to anonymous connections
    map to guest = bad user
 
+# Fruit global config
+   fruit:aapl = yes
+   fruit:nfs_aces = no
+   fruit:copyfile = no
+   fruit:model = MacSamba
+
+   inherit permissions = yes
+   multicast dns register = no
+
+# Protocol versions
+   client max protocol = default
+   client min protocol = SMB2_02
+   server max protocol = SMB3
+   server min protocol = SMB2_02
+
 
 ########## Domains ###########
 
@@ -240,3 +255,15 @@
 # Please note that you also need to set appropriate Unix permissions
 # to the drivers directory for these users to have write rights in it
 ;   write list = root, @lpadmin
+
+[myMacTimeMachine]
+   vfs objects = catia fruit streams_xattr
+   fruit:time machine = yes
+   fruit:time machine max size = 500G
+   comment = Time Machine backup
+   path = /data/MyMacTimeMachine
+   available = yes
+   valid users = kw
+   browseable = yes
+   guest ok = no
+   writable = yes
```

In my case, I set the section name as `myMacTimeMachine`.
I'm not sure what characters are safe to use, but alphanumeric characters should be okay.

## Setup avahi-daemon for Time Machine

Quoted from Johner's post above:

> Basically Samba shares the network drive and avahi makes it work with apple devices by implementing Apple’s Zeroconf architecture (also known as “Rendezvous” or “Bonjour”).

Install with:

```bash
sudo apt install avahi-daemon
```

Create an avahi config file with `sudo vim /etc/avahi/services/samba.service`:

```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_smb._tcp</type>
    <port>445</port>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>0</port>
    <txt-record>model=TimeCapsule8,119</txt-record>
  </service>
  <service>
    <type>_adisk._tcp</type>
    <txt-record>dk0=adVN=timemachine,adVF=0x82</txt-record>
    <txt-record>sys=waMa=0,adVF=0x100</txt-record>
  </service>
</service-group>
```

Be sure to fill the section name of `/etc/samba/smb.conf` in the first `<txt-record>`:

```diff
diff --git 1/samba.service 2/samba.service
index c516aac..5651005 100644
--- 1/samba.service
+++ 2/samba.service
@@ -13,7 +13,7 @@
   </service>
   <service>
     <type>_adisk._tcp</type>
-    <txt-record>dk0=adVN=timemachine,adVF=0x82</txt-record>
+    <txt-record>dk0=adVN=myMacTimeMachine,adVF=0x82</txt-record>
     <txt-record>sys=waMa=0,adVF=0x100</txt-record>
   </service>
 </service-group>
```

## Run the services

Reference: <https://ubuntu.com/tutorials/install-and-configure-samba>.

Update the firewall rules:

```bash
sudo ufw allow samba
```

Reference: <https://gist.github.com/davisford/5984768>.

Finally (re)start the services.

```bash
sudo service smbd restart
sudo systemctl restart avahi-daemon
```

These two commands should be rerun every time the `/etc/samba/smb.conf` config file is updated.

## Configure Time Machine on Mac side

Open `System Preferences > Time Machine > Select Disk`.
The samba shared folder `myMacTimeMachine` should now appear in the `Available Disks` list.
