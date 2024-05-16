# Kannel Installation On Centos 7

"Streamline your SMS gateway setup with our project for Kannel installation on CentOS 7. Kannel is a robust open-source solution for SMS and MMS messaging services. Our project provides a straightforward, step-by-step guide to easily configure and deploy Kannel on CentOS 7, enabling you to efficiently manage SMS communication for your applications or services. Improve your messaging capabilities and enhance communication with this user-friendly installation guide for CentOS 7."

## Table of Contents

- [Centos Update](#Update)
- [Dependencies](#Dependecies)
- [Kannel Download And Install](#Download)
- [Manual Start Kannel](#Manual)
- [Automatically Start Kannal](#Automatically)
- [Download And Install Nagios](#Nagios)
- [Enable Kannel Port](#EnablePort)
- [LOG File Rotation](#Rotation)
- [Database](#Database)
- [SMPP Connection Binding And Other Configuration](#Configuration)


### Update System:
----------------------

Note : Start by updating your package repositories and upgrading installed packages:
```sh
sudo apt update
sudo apt upgrade
```

# Dependecies
```sh
 yum install -y openssl-devel  libxml2-devel   texlive-*  m4  gcc-c++ make
 yum install -y openssl-devel  libxml2-devel   texlive-*  m4  gcc-c++ make
 cd usr/local
 curl -O https://ftp.gnu.org/gnu/bison/bison-2.7.tar.gz
 tar zxvf bison-2.7.tar.gz && cd bison-2.7 && ./configure && make && make install && cd src
 rm -rf  /usr/local/bin/bison 
 rm -rf /usr/bin/bison
 cp bison /usr/local/bin/bison && cp bison /usr/bin/bison
 cd .. && cd .. && rm -rf bison-2.7.tar.gz bison-2.7 
```
### Download
Download Kannel then build kannel project and finally install kannel.
```sh
cmd : curl -O -k https://www.kannel.org/download/1.4.5/gateway-1.4.5.tar.gz
cmd : tar -xvzf gateway-1.4.5.tar.gz
cmd : rm -rf gateway-1.4.5.zip (Remove)
cmd : cd gateway-1.4.5
cmd : ./configure   --prefix=/usr/local/gateway-1.4.5
cmd : make
cmd : make install
```

# Manual
Start Kannel Manually
```sh		
--start kannel
/usr/local/gateway-1.4.5/gw/bearerbox  /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
/usr/local/gateway-1.4.5/gw/smsbox  /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
```
Note:
The command '&> /dev/null &' is used to run a program in the background while discarding its output. The `&` at the end makes the command run in the background, and `>/dev/null` redirects both standard output and standard error to `/dev/null`, effectively silencing any output produced by the command. This is often used when you don't need to see or log the output of a background process.

*******************************************************************************************************************************
# Automatically
Automatically start kannel using cronjob.

## Step 1: Create File Like checkKannel
```sh
cmd : mkdir usr/local/scripts
cmd : cd usr/local/scripts
cmd : vi checkKannel
```	
- Then Paste Below Code in file and then save file
```sh
	if /usr/local/nagios/libexec/check_tcp localhost -p 1403 | grep "Connection refused"
	then
	#killall -9 bearerbox smsbox
	/usr/local/gateway-1.4.5/gw/bearerbox /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
	sleep 4
	/usr/local/gateway-1.4.5/gw/smsbox /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
	fi

	if /usr/local/nagios/libexec/check_tcp localhost -p 13013  | grep "Connection refused"
	then
	#killall -9 bearerbox smsbox
	/usr/local/gateway-1.4.5/gw/bearerbox /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
	sleep 4
	/usr/local/gateway-1.4.5/gw/smsbox /usr/local/gateway-1.4.5/gw/smskannel.conf &> /dev/null &
	fi
 ```
 
Note : above use port 1403 is admin port from config and 13013 is send sms user port from config file
	
- Give Permission To File For Run (below Command set this permission -rwxr-xr-x )
```sh

cmd : chmod 755 checkKannel
```
```sh
cmd : crontab -e
```
- Then paste below code in file
```sh	
	* * * * * usr/local/scripts/checkKannel
```
Note : The format is as follows:
```sh
* * * * * command-to-be-executed
- - - - -
| | | | |
| | | | +----- Day of the week (0 - 6) (Sunday = 0)
| | | +------- Month (1 - 12)
| | +--------- Day of the month (1 - 31)
| +----------- Hour (0 - 23)
+------------- Minute (0 - 59)
```
# Required Nagois for working cronjob code
Note:
Nagios is a monitoring system that keeps an eye on your computers and networks. It alerts you when things go wrong and lets you know when they get better. It helps organizations identify and resolve IT infrastructure issues, ensuring that critical systems are always up and running smoothly.

## Step 1 : System Update
```sh
cmd : sudo yum update
```
## Step 2 : Install Required Dependencies:
```sh
cmd : sudo yum install -y gcc glibc glibc-common wget unzip httpd php gd gd-devel perl postfix
```
## Step 3 : Create a Nagios User and Group:
```sh
cmd : sudo useradd nagios
cmd : sudo groupadd nagcmd
cmd : sudo usermod -a -G nagcmd nagios
```

## Nagios
Download and Compile Nagios Core:
```sh

cmd : wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz
cmd : tar -xzf nagios-4.4.6.tar.gz
cmd : cd nagios-4.4.6
cmd : ./configure --with-command-group=nagcmd
cmd : make all
cmd : sudo make install
cmd : sudo make install-commandmode
cmd : sudo make install-init
cmd : sudo make install-config
```

## Step 6 : Create a Nagios Web User Account:
```sh
cmd : sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
## Step 7 : Download and Install Nagios Plugins:
```sh

cmd : cd ~
cmd : wget https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
cmd : tar -xzf nagios-plugins-2.3.3.tar.gz
cmd : cd nagios-plugins-2.3.3
cmd : ./configure --with-nagios-user=nagios --with-nagios-group=nagios
cmd : make
cmd : sudo make install
```
## Step 8 : Start and Enable Apache and Nagios Services:
```sh

cmd : sudo systemctl start httpd
cmd : sudo systemctl enable httpd
cmd : sudo systemctl start nagios
cmd : sudo systemctl enable nagios
```


# EnablePort 
Enable Kannel Port For Send Sms On Kannel, Check kannel status etc. (If Sometime give error then reboot server then try again)
```sh
cmd : sudo systemctl status firewalld
cmd : sudo firewall-cmd --zone=public --add-port=1403/tcp --permanent
cmd : sudo firewall-cmd --zone=public --add-port=1505/tcp --permanent
cmd : sudo firewall-cmd --zone=public --add-port=13013/tcp --permanent
cmd : firewall-cmd --reload
```

# Rotation
Kannel Log File Rotation Setting
```sh
cmd : cd /etc/logrotate.d
cmd : vi kannel
```

	File Format
	/var/log/kannel/*.log {
              daily
              missingok
              rotate 31
              compress
              delaycompress
              notifempty
              create 640 root root
              sharedscripts
              postrotate
                    killall -HUP bearerbox smsbox || true > /dev/null 2> /dev/null
              endscript
	}

# Database
Create Database with name of [**kannel**] and get below database script For DLR Table
```sh
CREATE TABLE dlr (smsc varchar(40) DEFAULT NULL,ts varchar(50) DEFAULT NULL, source varchar(40) DEFAULT NULL,service varchar(40) DEFAULT NULL,url varchar(255) DEFAULT NULL,mask int(10) DEFAULT NULL,status int(20) DEFAULT NULL,boxc varchar(40) DEFAULT NULL,destination varchar(40) DEFAULT NULL,createDate timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,KEY smscts (smsc,ts)) ENGINE=MyISAM DEFAULT CHARSET=latin1; 
```
| Field        | Type          | Null | Key | Default            | Extra |
|--------------|---------------|------|-----|--------------------|-------|
| smsc         | varchar(50)   | YES  | MUL | NULL               |       |
| ts           | varchar(40)   | YES  |     | NULL               |       |
| source       | varchar(40)   | YES  |     | NULL               |       |
| service      | varchar(40)   | YES  |     | NULL               |       |
| url          | varchar(255)  | YES  |     | NULL               |       |
| mask         | int(10)       | YES  |     | NULL               |       |
| status       | int(20)       | YES  |     | NULL               |       |
| boxc         | varchar(40)   | YES  |     | NULL               |       |
| destination  | varchar(40)   | YES  |     | NULL               |       |
| createDate   | timestamp     | NO   |     | CURRENT_TIMESTAMP  |       |


# Configuration
Sample configuration file for binding smpp connections and so on. You will find this file in ** /usr/local/gateway-1.4.5/gw/smskannel.conf **.
```sh
# THIS IS A SAMPLE CONFIGURATION FOR SMS KANNEL
#
# This basic version is used for system testing. It expects 'fakesmsc' to
# send messages which are replied with simple fixed text message.
# It is run like this:
#
#  1% gw/bearerbox gw/smskannel.conf
#  2% gw/smsbox gw/smskannel.conf
#  3% test/fakesmsc -i 0.1 -m 100 "123 345 text nop"
#
# ..all 3 commands in separate shells (or screen sessions)
# Note that you can run them in different machines but have to
# add certain command line argument and configuration variables then
#
#
# For any modifications to this file, see Kannel User Guide
# If that does not help, see Kannel web page (http://www.kannel.org) and
# various online help and mailing list archives
#
# Notes on those who base their configuration on this:
#  1) check security issues! (allowed IPs, passwords and ports)
#  2) groups cannot have empty rows inside them!
#  3) read the user guide
#
# Kalle Marjola for Kannel project 2001, 2004

#---------------------------------------------
# CORE
#
# There is only one core group and it sets all basic settings
# of the bearerbox (and system). You should take extra notes on
# configuration variables like 'store-file' (or 'store-dir'),
# 'admin-allow-ip' and 'access.log'


group = core
admin-port = 1403
smsbox-port = 1505
admin-password = bar
#status-password = foo
#admin-deny-ip = ""
#admin-allow-ip = ""
log-file = "/var/log/kannel/kannel.log"
#log-level = 0
box-deny-ip = "*.*.*.*"
box-allow-ip = "127.0.0.1"
#unified-prefix = "+358,00358,0;+,00"
access-log = "/var/log/kannel/access.log"
log-level = 1
store-type = file
store-file = "/root/kannel.store"
#ssl-server-cert-file = "cert.pem"
#ssl-server-key-file = "key.pem"
#ssl-certkey-file = "mycertandprivkeyfile.pem"

#---------------------------------------------
# SMSC CONNECTIONS
#
# SMSC connections are created in bearerbox and they handle SMSC specific
# protocol and message relying. You need these to actually receive and send
# messages to handset, but can use GSM modems as virtual SMSCs


#TX=1 / 10 VJTxn
group = smsc
smsc = smpp
smsc-id = VJTxn
host = 46.4.70.222
port = 5555
smsc-username=demoaccount
smsc-password=lX7dtjF9
system-type=SMPP
transceiver-mode=1
#connect-allow-ip = 127.0.0.1
allowed-smsc-id = VJTxn
#validityperiod = 1440
#alt-charset = utf-8
max-pending-submits=20
throughput=30
#msg-id-type = 0x01

#TX=2 / 10 VJTxn
group = smsc
smsc = smpp
smsc-id = VJTxn
host = 46.4.70.222
port = 5555
smsc-username=demoaccount
smsc-password=lX7dtjF9
system-type=SMPP
transceiver-mode=1
#connect-allow-ip = 127.0.0.1
allowed-smsc-id = VJTxn
#validityperiod = 1440
#alt-charset = utf-8
max-pending-submits=20
throughput=30
#msg-id-type = 0x01


#----------Here You Can TLV Parameters Setting-------------------------

group = smpp-tlv
name = PE_ID
tag = 0x1400
type = octetstring
length = 50
smsc-id = VJTxn

group = smpp-tlv
name = Template_ID
tag = 0x1401
type = octetstring
length = 50
smsc-id = VJTxn

#------------


#---------------------------------------------
# SMSBOX SETUP
#
# Smsbox(es) do higher-level SMS handling after they have been received from
# SMS centers by bearerbox, or before they are given to bearerbox for delivery

group = smsbox
bearerbox-host = 127.0.0.1
sendsms-port = 13013
global-sender = 13013
#sendsms-chars = "0123456789 +-"
log-file = "/var/log/kannel/smsbox.log"
log-level = 4
access-log = "/var/log/kannel/access.log"
log-level = 4

#---------------------------------------------
# SEND-SMS USERS
#
# These users are used when Kannel smsbox sendsms interface is used to
# send PUSH sms messages, i.e. calling URL like
# http://kannel.machine:13013/cgi-bin/sendsms?username=tester&password=foobar...

group = sendsms-user
username = tester
password = foobar
max-messages = 25
concatenation = true
#user-deny-ip = ""
#user-allow-ip = ""

#---------------------------------------------
# SERVICES
#
# These are 'responses' to sms PULL messages, i.e. messages arriving from
# handsets. The response is based on message content. Only one sms-service is
# applied, using the first one to match.

group = sms-service
keyword = nop
text = "You asked nothing and I did it!"

# There should be always a 'default' service. This service is used when no
# other 'sms-service' is applied.

group = sms-service
keyword = default
text = "No service specified"
max-messages = 11
concatenation = true

#---------For Database Setting ---------------------------

group = mysql-connection
id = mydlr
host = localhost
username ="root"
password ="Vijay@123"
database = kannel
max-connections = 125

group = dlr-db
id = mydlr
table = dlr
field-smsc = smsc
field-timestamp = ts
field-destination = destination
field-source = source
field-service = service
field-url = url
field-mask = mask
field-status = status
field-boxc-id = boxc

``
