# raspbx_org
Welcome to RasPBX – Asterisk for Raspberry Pi

This project site maintains a complete install of Asterisk and FreePBX for the famous Raspberry Pi. Check the download page for the latest RasPBX image, which is based on Debian Buster (Raspbian) and contains Asterisk 16 and FreePBX 15 pre-installed and ready-to-go.
Read the documentation section about everything related to RasPBX in particular. It has been written for users with FreePBX experience, if you are still new please read the FreePBX website and documentation as well.

Active RasPBX installations are supported with updates:

FreePBX has it’s own update system, use Module Admin to keep FreePBX up to date.
Asterisk is supplied by RasPBX repositories, use raspbx-upgrade to get updates.
All other software packages on the system are supplied by the Raspbian project, raspbx-upgrade installs these updates as well.
On top of stock Asterisk / FreePBX a few additions have been included in RasPBX. They can be installed on demand with console based installer scripts:

Improved security with Fail2Ban
GSM VoIP gateway with chan_dongle
Backups
Fax gateway
In case you are stuck with your setup and have a question or you want to report a possible bug, please check out the RasPBX forum.

The RasPBX project is active since May 2012. Read about the project history on the blog.

========================================================================================================

Documentation
Contents – Basic Setup
Next steps after downloading the image
Determine hostname / IP address
Basic configuration
Email setup
Changing passwords
More documentation
Contents – Advanced Topics
Fax gateway
Security: HowTo for Asterisk and Fail2Ban
Running RasPBX without Internet connection
GSM VoIP Gateway with Chan_dongle
Security Considerations
Backup your System
Running RasPBX from an External USB HDD or Thumb Drive
Basic Setup
1. Next steps after downloading the image
You might want to have a look at Matej Kovacic’s excellent guide “RasPBX installation for beginners“, which explains the installation in great detail and also features a lot of additional useful tweaks.

Instructions on how to write the downloaded image to your card can be found here:
http://elinux.org/RPi_Easy_SD_Card_Setup

The image is only utilizing 4GB of your card, even if you bought a bigger one. On a bigger card, you can make more space available to your root partition by running on the console of your booted RPi:

raspi-config
Select the option Advanced Options – Expand Filesystem.
If you rather prefer to do this manually, one of the easiest ways is using GParted on Linux. Details can be found here:
http://elinux.org/RPi_Resize_Flash_Partitions#Manually_resizing_the_SD_card_using_a_GUI_on_Linux

2. Determine hostname / IP address
Once your RPi is booted, you need to know it’s hostname or IP address for ssh login or to open the web GUI. On Windows computers, you can just use the hostname raspbx to access your RPi.

SSH login:

ssh root@raspbx
Web GUI:

http://raspbx
On Macintosh, use raspbx.local instead:

ssh root@raspbx.local
Web GUI:

http://raspbx.local
In case this is not successful you can check your router’s DHCP client list, and search for the IP associated with the name raspbx.
If this is still not working out, you can always just connect an HDMI monitor and USB keyboard, log in to the console with user root, password raspberry, and run the command:

ifconfig
3. Basic configuration
After your RPi has booted successfully, log in either on the console or by ssh with user root and password raspberry. Follow these steps to complete the initial configuration:

Create new ssh host keys to have individual keys for every setup:

regen-hostkeys
After this step your ssh client will warn about a changed host key on your next ssh connect.

Choose your timezone:

configure-timezone
Configure locale settings:

dpkg-reconfigure locales
Configure keyboard settings (not needed when working with ssh only):

dpkg-reconfigure keyboard-configuration
4. Email setup
Email delivery from your RPi is needed if you plan to have voicemails sent to users by email. Email already works in the default configuration using Exim4 as MTA. By default, Exim is configured to directly send mails to the recipient MX hosts. This is however discouraged, as many email providers classify emails coming from dynamic IP addresses as spam. To avoid this, you need to set a smarthost. Unless you have an open SMTP server on your network that can be used as smarthost without authentication, you will need to specify SMTP authentication credentials as well. It is basically possible to use almost any publicly available freemailer as smarthost with the RPi. For Gmail, take a look at this detailed walk-through: https://wiki.debian.org/Exim4Gmail

Have username and password as well as SMTP hostname (sometimes also referred to as outgoing mail server) of the email account you are going to use ready. Run on the console:

dpkg-reconfigure exim4-config
On the first configuration page select “mail sent by smarthost; received via SMTP or fetchmail”. On the following pages just keep the default values by pressing enter, until you reach the page starting with “Please enter the IP address or the host name of a mail server…”. Here, enter the SMTP hostname of your email provider. Again, keep default values on the remaining pages.
Then, edit the file passwd.client by running:

nano /etc/exim4/passwd.client
Add your credentials at the bottom of this file in the following format:

SMTP_HOSTNAME:USERNAME:PASSWORD
In most cases, the SMTP hostname used in this file is identical to the hostname used as smarthost before. If email fails to work, specify the reverse lookup of your email provider’s SMTP host IP address here.

Some email providers also require you to use sender addresses identical to one of the public email adresses of your account. In this case, edit:

nano /etc/email-addresses
On the bottom of this file add:

root: your_email@someisp.com
asterisk: your_email@someisp.com
This configures the sender address of all outgoing mail to your_email@someisp.com.

Finally, to activate your configuration run:

update-exim4.conf
You can test your email setup with this command:

send_test_email your_email@someisp.com
A test email should reach your inbox shortly.

5. Changing passwords
Once you are done with basic installation, you might want to change the following important passwords to keep your setup safe. As long as your system is running with a private IP address behind a router with all ports closed, these passwords will only affect people trying to log in from inside your network, as no one can log in from outside anyway.

Change the password for SSH or console login with:

passwd
To change the FreePBX login select Admin – Administrators in FreePBX. On the right side of the page below Add User select admin. The password can be changed here.

There are 2 more passwords that should be changed. In FreePBX open Settings – Advanced Settings. Find the field Asterisk Manager Password and change this password. On the same page, search for User Portal Admin Password and change the password for the ARI administrator login as well.

6. More documentation
Further documentation on how to work with the FreePBX GUI can be found here:

https://wiki.freepbx.org/display/FOP

Advanced Topics
1. Fax gateway
Fax gateway support in RasPBX is provided by HylaFAX, an optional feature which needs to be installed manually by calling:

install-fax
After installation is complete, configure a fax extension at your choice. This can also be skipped and done later by calling:

add-fax-extension
This is configuring HylaFAX, Iaxmodem and FreePBX. An additional extension is added to FreePBX which can be used as inbound destination for your fax DID. Faxes to this extension will be emailed to the address specified during the add-fax-extension run.

It is possible to have as many fax extensions as required by calling add-fax-extension multiple times. Each extension can me mapped to a different email address, thus having several virtual fax machines with different recipients.

For sending faxes, any HylaFAX client such as Winprint HylaFAX or any other can be used.

The default user to connect to HylaFAX is root with empty password.

2. Security: HowTo for Asterisk and Fail2Ban
Fail2Ban can be installed easily by calling:

install-fail2ban
This installer includes all steps described by Razvan Turtureanu’s how-to for installing Fail2Ban with Asterisk on RasPBX. Read the complete tutorial in the forum. The last section other security tips gives a good overview on security in general, be sure to read this even if you don’t decide to install Fail2Ban.

3. Running RasPBX without Internet connection
If Internet connection is not continuously present or not present at all, 2 issues can appear that prevent calling between extensions:

A. On system boot, current time is obtained through NTP. Asterisk only starts after time has been set correctly, to avoid problems that have been seen in connection with a large time jump on the system. If Asterisk is started with wrong time first and time is properly set later, audio on calls can be seriously distorted. Thus, the boot scripts only start Asterisk after time has been set, and in setups without Internet connection Asterisk will not start by default. To overcome this, install fake-hwclock:

apt-get install fake-hwclock
It saves the time on shutdown and loads it again on reboot.

Update: Dnsmasq is installed and configured as described below with upgrade #10. The steps below are not needed if all the latest upgrades are installed.

B. Asterisk gets into trouble when DNS lookups fail, leaving an unstable system. This can be fixed by installing dnsmasq:

apt-get install dnsmasq
configure:

cd /etc
mv resolv.conf resolv.conf.dnsmasq
edit /etc/dnsmasq.conf, change this section

# Change this line if you want dns to get its upstream servers from
# somewhere other that /etc/resolv.conf
resolv-file=/etc/resolv.conf.dnsmasq
Then create /etc/resolv.conf with contents:

nameserver 127.0.0.1
Then reload:

/etc/init.d/dnsmasq restart

==================================================================================================

GSM VoIP Gateway with Chan_dongle

A highly affordable GSM VoIP gateway can be obtained using Huawei E155X or compatible USB modems and chan_dongle, providing both inbound and outbound calls on GSM/3G networks.

Hardware requirements
Chan_dongle is able to work with many different USB modems from Huawei, such as K3715, E169 / K3520, E155X, E175X, K3765 and others. Read the full compatibility list here.
Before connecting the modem to your RPi, be sure to use a power supply rated at least 1A, better 1.2A or more. If your power supply does not provide enough current to power both the RPi and the modem, you can get problems booting the RPi, or calls to/from the GSM network fail with error “dongle disconnected”.
Alternatively a powered USB hub can be used as well. If you want to use 2 modems, an additional WiFi connection or other devices that require considerable amount of current, a powered USB hub is absolutely required.
In case you are using one of the older 256MB models with F1 and F2 still in place, you might want to consider bridging over F1 and F2, read more details here. This is however not required on the newer 512MB models, or when using a powered USB hub.

Not all Huawei USB modems work out of the box, on some of them voice calling capability has to be enabled first, some need to be upgraded with the latest firmware. Details on this can be found on the original chan_dongle wiki.
Before inserting the SIM into your modem please deactivate the PIN on your card. This can be done with any phone. Insert the SIM into your phone, deactivate PIN and you’re done. You can also use the Mobile Partner software – which comes with the modem itself – on a Windows computer to do this.

Setup and configuration
Once your modem has PIN deactivated, latest firmware and voice enabled, run this command:

 install-dongle
This installer script installs chan_dongle.so, and creates an initial configuration. The script is provided with upgrade #11 (and improved further with upgrade #12). Once the installer has finished, connect your modem to the RPi. If it was connected already before, unplug it now and plug it in again. Some older modems still require usb_modeswitch to enable data/audio mode, for those a complete reboot of the RPi is recommended at this point.

Then log into FreePBX, in Connectivity – Trunks click Add Custom Trunk. Provide a trunk name, set Outbound CallerID to the number of your SIM, and enter in the field Custom Dial String:

dongle/dongle0/$OUTNUM$
Add an outbound route to use this trunk, as well as an incoming route. On the incoming route set DID Number to your SIM number, precisely matching the number you entered when running the install-dongle script.

More details on how to setup inbound and outbound routes can be found in the forum.

Sending and receiving SMS
The install-dongle script provides a few basic options to send and receive SMS. Received messages can be forwarded by email. If no email address is specified, messages are stored in /var/log/asterisk/sms.txt instead.
Additionally, received messages can optionally be forwarded to a mobile phone number on top of sending them by email. This is done through the first dongle0, your mobile operator’s charges apply for sending SMS.
In order to send out custom messages, a password protected web page can be activated during the install-dongle run. This page is located at http://raspbx/sms or http://raspbx.local/sms (for Mac).
A web page for sending USSDs is optionally available. Install it with:

apt-get install ussd-webpage
The page can be found at http://raspbx/ussd or http://raspbx.local/ussd (for Mac).

Troubleshooting
If it doesn’t work at this point, at first check if the interfaces of your modem match the configuration:

ls -l /dev/tty*
2 devices, ttyUSB1 and ttyUSB2 should show up. If the numbers are different, something like ttyUSB0 and ttyUSB1, edit /etc/asterisk/dongle.conf and change the values below [dongle0] accordingly.
Many modems require the usb_modeswitch program to switch over from CD-drive mode (for installing Windows drivers) to modem mode. The /dev/ttyUSB devices will not appear before the switch-over was done. Make sure to have all the latest upgrades on recent Debian Jessie based images to fix some initial bugs with usb_modeswitch. If your modem is still not switching it might be necessary to add a custom rule for usb_modeswitch.

Some users have reported voice calling with a specific mobile operator was not working, while using a sim card from a different provider with the same modem and RPi worked perfectly fine.

Troubleshooting power problems
In case the modem devices ttyUSB1 and ttyUSB2 are disappearing as soon as a call is set up (or also at other times), and later they appear again, it is most likely the modem is not sufficiently powered and thus disconnects again and again. Try these options:

Remove any USB extension cord from the modem and connect it directly.
In case a powered USB hub is used, try a different one or even try without it. With some of those hubs the modems just don’t work properly, although the current rating of the hub’s power supply is high enough.
If the modem is directly connected to the RPi (without a hub), the power supply has to power them both. Try different power supplies, as some have problems to keep their output voltage stable under higher load.
As a last resort, you can also try to use a different modem.
For more information, read the complete documentation here:

https://github.com/bg111/asterisk-chan-dongle/wiki

Modems reported working/not working
On top of the compatibility list on the original chan_dongle wiki, users of RasPBX have reported several modems to work fine with the RPi:

E153
E1550
E1552
E156G
E160
EG162
E166
E169
E171
E173 (some types of E173 seem to not work, only E173 with Qualcomm chipsets do work)
E1750
E180
E303
K3520 (not to confuse with K3520-z)
K3715
K3765a
The following modems had issues and could not me made working so far. Please let us know if they work for you nonetheless:

E150
E1752
E303C
E352
K3520-z

==================================================================================================

Downloads
Support for Raspbian 11 Bullseye:
Please have a look at Ronald Raikes’ project who also supports Asterisk and FreePBX for the Raspberry Pi:

FreePBX 15 or 16 plus Asterisk 15, 16, 17, 18, or 19 on a Raspberry Pi with Buster or Bullseye:

https://www.dslreports.com/forum/r30661088-PBX-FreePBX-for-the-Raspberry-Pi

Many thanks to Ronald who continuously updated his project over the years. Due to personal reasons it is unfortunately currently not possible to update the original RasPBX again in order to support the latest Pi 4 hardware revisions.

RasPBX images based on Raspbian 10 Buster:
The latest image supports Pi 4, Pi 3 and Pi 2 (Pi 1 and Pi zero no longer supported):

Torrent	raspbx-10-10-2020.zip.torrent
HTTP	raspbx-10-10-2020.zip
SHA-1	b36d0cc9c4986611376df6ce5141cdc3ee296540
Contents	Asterisk 16.13.0 & FreePBX 15.0.16.75
An 8GB card or larger is recommended.

Upgrades:
Once your RasPBX has successfully booted, run this command on the console to install the latest additions and improvements (see upgrades list):

raspbx-upgrade
Notes:

FreePBX 15 is still in beta stage at the time of this image release. It is recommended to enable “Set Module Admin to Edge mode” in Settings – Advanced Settings and then update all FreePBX modules from the edge track.
In case the “Apply Config” button takes very long or never completes, set “Enable Module Signature Checking” to no in Settings – Advanced Settings.
Pi 1 and Pi zero models are no longer supported. Please refer to the previous Stretch based images for Pi 1 and Pi zero hardware.
Pi 4 with 8GB RAM hangs when running the upgrades. Pi 4 with 4GB or 2GB RAM is working fine.
Changes:

Upgrade to FreePBX 15
Upgrade to Raspbian Buster, supported for 3 years from now
Based on official Raspbian Buster Lite
SSH login:
user: root
password: raspberry

Older versions based on Raspbian Buster:

Torrent	raspbx-11-11-2019.zip.torrent
HTTP	raspbx-11-11-2019.zip
SHA-1	3d403f60c9f08fae96de0a7dbc7df6b7834d84df
Contents	Asterisk 16.6.1 & FreePBX 15.0.16.22
Torrent	raspbx-06-10-2019.zip.torrent
HTTP	raspbx-06-10-2019.zip
SHA-1	a1e19a5c988e19ce85104d22b57621d1e95c1f06
Contents	Asterisk 16.5.0 & FreePBX 15.0.16.19
RasPBX images based on Raspbian 9 Stretch:
This image supports Pi 3 B+, Pi 3, Pi 2, B+, B and A models with the following improvements and overall changes compared to the previous Raspbian Jessie based images:

Upgrade to FreePBX 14 with many improvements and new features. Read all details on the FreePBX blog.
Based on Raspberry Pi Foundation’s official Raspbian Stretch. Stretch ships with major package upgrades and changes, such as MariaDB which is replacing MySQL. MariaDB has better performance than MySQL, RasPBX users should notice a faster GUI compared to previous releases.
PJSIP is now the default SIP stack listening on port 5060. Chan SIP is still available on port 5160.
Zram replaces disk-based swap.
Torrent	raspbx-04-04-2018.zip.torrent
HTTP	raspbx-04-04-2018.zip
SHA-1	8f473d01935da0347fbafb7f71c649914934c5b6
Contents	Asterisk 13.20.0 & FreePBX 14.0.2.10
A 4GB card is required.

Upgrades:
Once your RasPBX has successfully booted, run this command on the console to install the latest additions and improvements (see upgrades list):

raspbx-upgrade
Since the image posted on 28/01/2017 these fixes and improvements have been made:

Upgrade to FreePBX 14 (latest stable release)
Upgrade to Raspbian Stretch, supported for 3 years from now
Based on official Raspbian Stretch Lite 2017-09-07 image
MariaDB replaces MySQL
Zram replaces disk-based swap
SSH login:
user: root
password: raspberry

Older versions based on Raspbian Stretch:
raspbx-03-12-2017.zip.torrent
SHA-1: 7d04b2087fbb4c9203013e5c783a79b5f348e05f
Date: December 3rd 2017
Contents: Asterisk 13.18.3 & FreePBX 14.0.1.20

raspbx-10-10-2017.zip.torrent
SHA-1: 783f85f9094c7d852aa08d70e640b7b51b687ee6
Date: October 10th 2017
Contents: Asterisk 13.17.1 & FreePBX 14.0.1.8

RasPBX images based on Raspbian 8 Jessie
Login details for all images below (user / password):
SSH login: root / raspberry
Default FreePBX login: admin / admin
Mysql root password: raspberry

Torrent	raspbx-28-01-2017.zip.torrent
HTTP	raspbx-28-01-2017.zip
SHA-1	f1721571a0c0dbdec9ec599af2453d719eea52a9
Contents	Asterisk 13.13.1 & FreePBX 13.0.190.11
A 4GB card is required.
Changes:

RasPBX Upgrades #1 – #24 included (see list)
Including support for Pi 3, Pi 2, B+, B and A models
Latest kernel 4.4.38+ and all Raspbian updates
Note: When booting this image on a Pi 1 it usually takes around 5 minutes for Asterisk to start. On the Pi 2 and Pi 3 Asterisk starts after approx. 1 minute.

Older versions based on Raspbian Jessie:

raspbx-22-09-2016.zip.torrent
SHA-1: bca50fd1d841b1292ebcaa78a1d996ee70b8ae04
Date: September 22nd, 2016
Contents: Asterisk 13.11.2 and FreePBX 13.0.188.8
Changes:

Shipping with Asterisk 13 by default
RasPBX Upgrades #1 – #24 included (see list)
Including support for Pi 3, Pi 2, B+, B and A models
Latest kernel 4.4.13+ and all Raspbian updates
raspbx-06-03-2016.zip.torrent
SHA-1: 38665490e4ca3771ef39623c72019737d979e988
Date: March 6th, 2016
Contents: Asterisk 11.21.0 and FreePBX 13.0.74
Changes:

Added support for Pi 3
Based on official Raspbian Jessie Lite 2015-11-21 image
RasPBX Upgrades #1 – #21 included (see list)
Including support for Pi 3, Pi 2, B+, B and A models
Latest kernel 4.1.18+ and all Raspbian updates
raspbx-25-01-2016.zip.torrent
SHA-1: f1c24f3f2738e97cab46b716ac32dc0cb4d9e866
Date: January 25th 2016
Contents: Asterisk 11.21.0 and FreePBX 13.0.51
Changes:

Upgrade to FreePBX 13 (latest stable release)
Based on official Raspbian Jessie Lite 2015-11-21 image
RasPBX Upgrades #1 – #21 included (see list)
Including support for Pi 2, B+, B and A models
Latest kernel 4.1.15+ and all Raspbian updates
raspbx-17-10-2015.zip.torrent
SHA-1: f376767fcc71280ac18b82c42ea5896a09ca22c6
Date: October 10th 2015
Contents: Asterisk 11.20.0 and FreePBX 12.0.76.2
Changes:

RasPBX Upgrades #1 – #20 included (see list)
Improved SD card I/O performance
Including support for Pi 2, B+, B and A models
Latest kernel 4.1.10+ and all Raspbian updates
Note: SD card corruption can appear on a Pi 2
raspbx-22-02-2015.zip.torrent
SHA-1: 69ece60cd2d46813e69a8abb1a148057fde82a5d
Date: February 22nd 2015
Contents: Asterisk 11.15.0 and FreePBX 12.0.38
Changes:

Upgrade to Debian Jessie (supported for 3 years from now on)
Upgrade to FreePBX 12 (latest stable release)
RasPBX Upgrades #1 – #19 included
Including support for Pi 2, B+, B and A models
Latest kernel 3.18.6+ and all Raspbian updates
Note: SD card corruption can appear on a Pi 2
raspbx-09-02-2015.zip.torrent
SHA-1: 42758916b6816d4ac5743343fd2dec3d5d06b8be
Date: February 9th 2015
Contents: Asterisk 11.15.0 and FreePBX 2.11.0.42
Changes:

Including support for Pi 2, B+, B and A models
Upgrade to Debian Jessie (supported for 3 years from now on)
Upgrades #1 – #19 included
Latest kernel 3.18.6+ and all Raspbian updates
Note: SD card corruption can appear on a Pi 2
RasPBX images based on Raspbian 7 Wheezy
Login details for all images below (user / password):
SSH login: root / raspberry
Default FreePBX login: admin / admin
Mysql root password: raspberry

raspbx-31-07-2014.zip.torrent
SHA-1: 69d5c2eeda7b0b845931bb38e2807f2ef757481b
Date: July 31st 2014
Contents: Asterisk 11.11.0 and FreePBX 2.11.0.38
Changes:

Upgrades #1 – #18 included
Including support for B+ models
Latest kernel and all Raspbian updates
Last image based on Debian Wheezy
raspbx-14-02-2014.zip.torrent
SHA-1: dc1dc8cf91cfda15456625b730999abe88f8f366
Date: February 14th 2014
Contents: Asterisk 11.6.0 & FreePBX 2.11.0.23
Changes:

Upgrades #1 – #17 included
Latest FreePBX security fixes included
Latest kernel 3.10.29+ and all Raspbian updates
raspbx-12-08-2013.zip.torrent
SHA-1: d451501323a2c169f6c6c00c217be9d8846b361b
Date: August 12th 2013
Contents: Asterisk 11.5.0 & FreePBX 2.11.0.10
Changes:

Upgrades #1 – #14 included
Latest kernel 3.6.11+ and all Raspbian updates
raspbx-27-04-2013.zip.torrent
SHA-1: 97afba03ad80c2b89dee7d427424ce355e64643c
Date: April 27th 2013
Contents: Asterisk 11.3.0 & FreePBX 2.11.0.0rc1.2
Changes:

Upgrades #1 – #11 included
Latest kernel 3.6.11+ and all Raspbian updates
Installed additional FreePBX modules: Asterisk Logfiles, Announcements, Conferences, IVR, Queues, Ring Groups, Motif (Google Voice)
Fixed issues with symlinked config files due to recent changes in FreePBX
Installed sox and mpg123 for MOH conversion
Increased PHP upload max filesize and max post size to 20MB
raspbx-19-01-2013.zip.torrent
SHA-1: 14a76ae4c3eac0a46e8caf77cc6dad68c75b1a9c
Date: January 19th 2013
Contents: Asterisk 11.1.2 & FreePBX 2.11.0.0beta2.2
Changes:

Upgrades #1 – #5 included
Latest kernel 3.6.11+ and all Raspbian updates (improved speed!)
Upgraded to Asterisk 11 / FreePBX 2.11
Deleted some obsolete files –> smaller image zip file
raspbx-04-11-2012.zip.torrent
SHA-1: 6517759a7b4c899d25b3ab3d1d1e1b44379afbda
Date: November 4th 2012
Contents: Asterisk 1.8.15.1 & FreePBX 2.10.1.2
Changes:

Upgrades #1 – #4 included
Moved swap partition to swap file
Upgraded backup & restore module
raspbx-12-09-2012.zip.torrent
SHA-1: 303d178a84d28c8437ae8282f2dc59fbd910a91c
Date: September 12th 2012
Contents: Asterisk 1.8.15.1 & FreePBX 2.10.0
Changes:

Raspbian (Debian Wheezy) based
Latest kernel 3.2.27+ included
Additional kernel modules (iptables, ppp_mppe and others) included
Email delivery with Exim4
Watchdog enabled: automatic reboot after serious crash
Automatic update of ssh host keys on first boot
Startup script bugfix: console appears also without ntp
RasPBX images based on Raspbian 6 Sqeeze
debian6-asterisk-14-07-2012.zip.torrent
SHA-1: 931407a3eac4360f2cff074a86124d68c74b2441
Date: July 14th 2012
Contents: Asterisk 1.8.14.0 & FreePBX 2.10.0
Changes:

fixed CDR reports
added additional sounds from core sounds, music on hold and extra sounds
removed unused packages (xserver, etc.)
improved startup script: Asterisk only starts after correct system time has been obtained through ntp
added log file rotation for Asterisk and FreePBX logs
The first image published on June 2nd 2012 is no longer available for download.
