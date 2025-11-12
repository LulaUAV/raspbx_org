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
