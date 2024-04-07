# BKD-Linux-Server-Project
Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    - DHCP (one scope serving the local internal network)  isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on  separate disk, this partition must be mounted for the backup, then unmounted

2. One workstation running a desktop environment and the following apps:
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required** 
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk 
    - **Optional**
        1. Propose and implement a solution to remotely help a user


## Let's start off with the server part.
Using VirtualBox, create a VM with ubuntu server 22.04. Download ISO from [link](https://ubuntu.com/download/server)   
With the following configuration :
- 2048MB of RAM
- 25GB of storage
- 1 CPU
- network set to **Bridged Adaptater.**

------

  launch and follow the on screen instruction and don't forget to allow install open SSH Server.
  ~~darko~~ for username and ~~darkosrvr~~ for localhost name and ~~darkoroot~~ for password. 
    
  Now lets jump in the configuration. 
  
## DHCP service.
install with :
```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install isc-dhcp-server
```

go to -> `sudo nano /etc/dhcp/dhcpd.conf`
and do : 
 ```subnet 192.168.1.0 netmask 255.255.255.0 {
 range 192.168.1.6 192.168.1.66;
 option routers 192.168.1.254;
 option broadcast-address 192.168.1.255;
 option domain-name-servers 192.168.1.1, 192.168.1.2;
```

go to ->
```
sudo nano /etc/default/isc-dhcp-server
```
and do : 
```
INTERFACESv4="enp0s3"
```

run : 
```
sudo systemctl restart isc-dhcp-server.service
sudo systemctl enable isc-dhcp-server.service
sudo systemctl status isc-dhcp-server.service
```

which will give :

![dhcpstatus](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/e13b408c-0d48-4ba2-b352-714301fb0b1e)   

## DNS service : 
The well known bind9 DNS service is used.
do : 
```
sudo apt install bind9
```
go to -> `cd /etc/bind` then do -> `sudo nano named.conf.options` 
add forwarders 1.1.1.1 and 8.8.8.8    
Follow syntax given by the image below.

![darkoDNSsetting](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/7d6286bb-9c23-4745-a939-38d4dabca3cb)   

test it out by sending 4 ICMP requests to the specified host :
 ```
ping -c 4 x.x.x.x
```

which will give :

 ![darkopingDNStest](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/c562e39a-20a2-4e0e-be02-1ede902460dd)   

woohoo ! (for now .. ?)

do :
```
systemctl disable systemd-resolved
```
go to -> `cd /run/systemd/resolve/` then -> `sudo nano stub.resolv.conf`   
and modify nameserver to 127.0.0.1
 ```
systemctl restart bind9
```
 ```
systemctl status bind9
```
   
which will give   
![darkoDNSstatus](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/5a1628c3-0f61-4cfa-bdef-235ca97ab19b)   

and done. 

------

## Now the GLPI MariaDB Apache2 PHP config.
do for mariadb & Apache2 : 
```
sudo apt install mariadb-server
sudo mysql_secure-installation
```

To create the DB with its user info :
  ```
sudo mysql -u root -p
  CREATE DATABASE glpi;
  CREATE USER 'USERNAME'@'SERVERNAME' IDENTIFIED BY 'PASSWORD';
  GRANT ALL PRIVILEGES ON glpi.* TO 'USERNAME'@'SERVERNAME';
  FLUSH PRIVILEGES;
  EXIT;
```
here username is darko servername is darkosrvr and password is darko _(for originality)_
Now install PHP modules & Apache 2 :
```
sudo apt -y install php php-{curl,zip,bz2,gd,imagick,intl,apcu,memcache,imap,mysql,cas,ldap,tidy,pear,xmlrpc,pspell,mbstring,json,iconv,xml,gd,xsl}
```
```
sudo apt -y install apache2 libapache2-mod-php
```
!! ADD httponly flag to cookie -> `$ sudo vim /etc/php/*/apache2/php.ini
session.cookie_httponly = on` use ctrl+w to search for it.

Now now now, install GLPI 
```
wget https://github.com/glpi-project/glpi/releases/download/$VER/glpi-$VER.tgz
```
```
tar xvf glpi-$VER.tgz
```
!! move GLPI to /var/www/html dir. -> `sudo mv glpi /var/www/html/`   
Change ownership to Apache with recursive mode -> `sudo chown -R www-data:www-data /var/www/html/`   
IT IS TIME.
TO TEST AND FINISH.
THE GLPI INSTALL.

go to IP_SERVER `xxx.xxx.xxx.xxx/glpi/` on browser. (here it is 192.168.1.33)   
![GLPI SETUP](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/1eace066-20bc-4ff1-a006-2383e0739891)   

connect with IP of mariadb, USERNNAME@LOCALHOST and PASSWORD that you configured earlier. 

if you forgot IP do : `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf` it will be written next to "bind-address".

: ![step1SUCCESS](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/304be4b7-a0dd-46fc-8eb6-7da1a6a2a472)   

When you reach step 6 you will be given default credentials, save them. 

![step6](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/3a087b44-b1a2-45af-8ef6-f20eaaaad4ea)   

Once inside change the credentials for glpi/glpi 

![CRED](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/e9fa7735-c3a8-484e-b950-4516e0e8da11)   

Here Login is replaced to darko and password to dark0. If you don't know the person whose identity is used, look it up:) 

Okay you're done. _Are you still alive?_ Normally yes. 


------


## For The Firewall Config : 

do : 
```
sudo ufw enable`
```
add the ports that are of interest to you with `sudo ufw allow port_number/protocol`.   
For darko@darkosrvr, run `sudo ufw status` which will display the configured ports :   
 
![UFWports](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/d059132a-b5dd-480a-92c4-3d7808787521)   

------
## PART 2 Installing a workstation. 
The chosen OS for the workstation, is Mint 21.3. (because of nostalgia) from [here](https://www.linuxmint.com/download.php)   
it is also installed through VirtualBox.
with the following configuration : 
- 2048MB of RAM
- 30GB of storage
- 2 CPUs
- network to bridged Adaptater.

------
  ![mint01](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/8eb6f46c-8d80-4b7f-816b-4299d8438801)   
Steps are : 
1. select language
2. select keyboard
3. select multimedia codecs
4. PARTITION TIME to have /home folder on different partition but same disk.   
 ![finalpartitionMintLinux](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/39b6d162-398f-4464-bcd8-4acc44af0f43)      

/home folder needs at least 10GB of space for installation to continue.    
5. Select time zone a.k.a where you   
6. create user a.k.a who are you    
7. finalize and finish.    

------

We need to install LibreOffice, Gimp, Mullvad browser.
do : 
```
sudo apt update && sudo apt upgrade
```
```
sudo apt install libreoffice
```
```
sudo apt install gimp
```
for Mullvad Browser do : 
```
wget --content-disposition https://mullvad.net/en/download/browser/linux-x86_64/latest -P ~/Downloads
```
then :   
![mullvad install](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/bfb9440e-2b98-40e7-8ebf-b7d12fcef8bc)   

_(yes it is mint@mint, username and localhost are changed later on.)_   

Lets do an SSH test, since it was enabled during the ubuntu server installation no package had to be installed.    
Do from workstation : ssh@darkosrvr and see :

![sshTEST](https://github.com/marodorse/BKD-Linux-Server-Project/assets/34199422/e327508c-3283-42ef-a0f1-598baecdd6a7)

congratulations my friend.

