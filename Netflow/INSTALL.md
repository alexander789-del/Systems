These instructions install nfsen-ng on a fresh Ubuntu 22.04/24.04 LTS 
# Installation

```bash
# Use the sudo user
sudo -i

# Update ubuntu:
apt-get update
apt autoremove

# Create and navidate to a new folder:
mkdir ~/nfsen && cd ~/nfsen
or
mkdir /var/nfsen && cd /var/nfsen

# Download the required files:
wget https://bit.ly/2NpMHqV     >>> NfSend
wget https://github.com/phaag/nfdump/archive/v1.6.17.tar.gz    >>>NfDump

# Extract the files:
tar zxfv 2NpMHqV
tar xzfv v1.6.17.tar.gz

## Install Dependencies
apt install make gcc flex rrdtool librrd-dev libpcap-dev php librrds-perl libsocket6-perl apache2 libapache2-mod-php libtool dh-autoreconf pkg-config libbz2-dev byacc doxygen graphviz librrdp-perl libmailtools-perl build-essential autoconf

```bash
# you may use only >>> this is Important

apt-get install apache2 php libapache2-mod-php librrds-perl librrdp-perl librrd-dev libmailtools-perl build-essential autoconf rrdtool

Make sure the right version of PHP is being used:

a2enmod php7.4

Fix problem with displaying icons in nfsen:

Nano  /etc/apache2/mods-enabled/alias.conf

and comment out line: 'Alias /icons/ "/usr/share/apache2/icons/"

In the php.ini file, be sure to specify the correct time zone, for example:

nano /etc/php/7.4/apache2/php.ini

date.timezone = Asia/karachi

 

Prepare nfdump for compilation:

cd nfdump-1.6.17/

sh ./autogen.sh

./configure --enable-nsel --enable-nfprofile --enable-sflow --enable-readpcap --enable-nfpcapd --enable-nftrack --enable-jnat

Compile and install nfdump

make && make install

 

(it may be necessary to run /sbin/ldconfig or ldconfig as root after the installation)

 

Install nfsen dependencies:

cpan App::cpanminus

cpanm Mail::Header

cpanm Mail::Internet

 

Check the nfdump version:

nfdump -v

Configure nfsen:

cd ../nfsen-1.3.6p1  or   cd /nfsen-1.3.6p1/etc/

cp nfsen-dist.conf nfsen.conf

nano ./etc/nfsen.conf

you may changed the following (apach2 and nginx work by default from the www-data user):

>
# user and group of the web server process
# All netflow processing will be done with this user

$BASEDIR = "/var/nfsen";

$PREFIX  = '/usr/local/bin';

$USER    = "netflow";

$WWWUSER  = "www-data";
$WWWGROUP = "www-data";

# number of nfprofile processes to spawn during the profiling phase
# depends on how busy your system is and how many CPUs you have
# on very busy systems increase it to a higher value
#$PROFILERS = 2;

Add user used by nfsen:

useradd -M -s /bin/false -G www-data netflow

 

Create nfsen base directory:

mkdir – p /var/nfsen

 

Install nfsen:

./install.pl ./etc/nfsen.conf

 

If there is a version mismatch change this:

nano libexec/NfSenRRD.pm

Change from 1.5 t0 1.8

 

Point default Apache site to nfsen.php file:

nano /etc/apache2/sites-enabled/000-default.conf

DocumentRoot /var/www/nfsen

DirectoryIndex nfsen.php

ServerAdmin webmaster@localhost

        ServerName 192.168.88.163

        DocumentRoot /var/www/nfsen
        DirectoryIndex nfsen.php)

 

change apache port 80 to any_port_number

 

nano /etc/apache2/apache2.conf

systemctl enable apache2

systemctl start apache2

Start nfsen service:

/var/nfsen/bin/nfsen start

 

Restart Apache:

systemctl apache2 restart

 

If you need to run nfsen on port 2055/udp, and it was taken by default by nfdump (by the nfcapd process), then stop it before running nfsen:

systemctl is-enabled nfdump

systemctl stop nfdump

netstat -anpl | grep 2055

kill -9 PID_NUMBER

netstat -anpl | grep nfcapd

Testing >>> on Mikrotik

nano /var/nfsen/etc/nfsen.conf
%sources = (
' MikroTik_CCIE '    => { 'port' => '2055', 'col' => '#00ff00', 'type' => 'netflow' },
#    'upstream1'    => { 'port' => '9995', 'col' => '#0000ff', 'type' => 'netflow' },
#    'peer1'        => { 'port' => '9996', 'IP' => '192.168.88.1' },
#    'peer2'        => { 'port' => '9996', 'IP' => '0.0.0.0' },
);

 #OR

%sources = (
      'source1' => { 'port' => '9995', 'col' => '#0000ff', 'type' => 'netflow' },
      'source2' => { 'port' => '9996', 'col' => '#cc3333', 'type' => 'netflow' },
      'source3' => { 'port' => '9997', 'col' => '#99ff33', 'type' => 'netflow' },
  );

# OR

%sources = (
    'ccr1016'      => { 'port' => '9995', 'IP' => 'x.x.x.x', 'col' => '#0000ff', 'type' => 'netflow' },
    'apfloor1'     => { 'port' => '9995', 'IP' => 'x.x.x.x', 'col' => '#8B0000' },
    'apfloor2'     => { 'port' => '9995', 'IP' => 'x.x.x.x', 'col' => '#DC143C'},
    'apfloor3'     => { 'port' => '9995', 'IP' => 'x.x.x.x', 'col' => '#FF7F50'},
);
 

 

/etc/init.d/nfsen reconfig

Or

cd /var/nfsen/bin

./nfsen reconfig

./nfsen start

sudo /etc/init.d/nfsen reconfig
 

To make nfsen reboot proof:

ln -s /var/nfsen/bin/nfsen  /etc/init.d/nfsen

update-rc.d nfsen defaults 20

It remains to configure the web server or just create a symbolic link in the www directory (after that you can open nfsen in a browser, for example http://192.168.88.157/nfsen/nfsen.php):

               

ln -s /var/nfsen/www/ /var/www/html/nfsen

 

ln -s /var/www/nfsen/ /var/www/html/nfsen

 

Make sure that nfsen starts when the operating system starts:

systemctl enable nfsen

systemctl start nfsen

systemctl status nfsen

Browse to:

http://yourip:portnumber

Auto start at boot
Create /etc/systemd/system/nfsen.service:
[Unit]
Description=NfSen Service
After=network.target

[Service]
Type=forking
PIDFile=/var/nfsen/var/run/nfsend.pid
ExecStart=/var/nfsen/bin/nfsen start
ExecStop=/var/nfsen/bin/nfsen stop
Restart=on-abort

[Install]
WantedBy=multi-user.target
 



Troubleshooting commands:

sudo netstat -tulpn

ls -l /var/nfsen/profiles-stat/live

timedatectl set-timezone Asia/karachi

chmod -R 777 /var/nfsen/var/run/nfsen.comm

chown -R www-data:www-data /var/nfsen

chown -R netflow:www-data /var/nfsen/profiles-data/live/

tcpdump port 2055 -e -n
