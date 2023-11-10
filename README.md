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

#Database
Create Database with name of kannel and get below database script For DLR Table
```sh
CREATE TABLE dlr (smsc varchar(40) DEFAULT NULL,ts varchar(40) DEFAULT NULL, source varchar(40) DEFAULT NULL,service varchar(40) DEFAULT NULL,url varchar(255) DEFAULT NULL,mask int(10) DEFAULT NULL,status int(20) DEFAULT NULL,boxc varchar(40) DEFAULT NULL,destination varchar(40) DEFAULT NULL,createDate timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,KEY smscts (smsc,ts)) ENGINE=MyISAM DEFAULT CHARSET=latin1; 
```
+-------------+--------------+------+-----+-------------------+-------+
| Field       | Type         | Null | Key | Default           | Extra |
+-------------+--------------+------+-----+-------------------+-------+
| smsc        | varchar(40)  | YES  | MUL | NULL              |       |
| ts          | varchar(40)  | YES  |     | NULL              |       |
| source      | varchar(40)  | YES  |     | NULL              |       |
| service     | varchar(40)  | YES  |     | NULL              |       |
| url         | varchar(255) | YES  |     | NULL              |       |
| mask        | int(10)      | YES  |     | NULL              |       |
| status      | int(20)      | YES  |     | NULL              |       |
| boxc        | varchar(40)  | YES  |     | NULL              |       |
| destination | varchar(40)  | YES  |     | NULL              |       |
| createDate  | timestamp    | NO   |     | CURRENT_TIMESTAMP |       |
+-------------+--------------+------+-----+-------------------+-------+
