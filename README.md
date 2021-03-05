yum -y install httpd php gcc glibc glibc-common wget perl gd gd-devel unzip zip

Create a nagios user and nagcmd group for allowing the external commands to be executed through the web interface, add the nagios and apache user to be a part of the nagcmd group.

useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
usermod -a -G nagcmd apache

Install Nagios Server
Download the latest version of Nagios Core using the terminal.

cd /tmp/
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz
tar -zxvf nagios-4.4.3.tar.gz
cd /tmp/nagios-4.4.3

Compile and Install Nagios.

./configure --with-nagios-group=nagios --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode

Install Nagios Web Interface
Install the Nagios web configuration using the following command.

make install-webconf

Run the following command to install a Nagios exfoliation theme

make install-exfoliation

Create a user account (nagiosadmin) for logging into the Nagios web interface. Remember the password that you assign to this user – you’ll need it later.

htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
Restart Apache web server to make the new settings take effect.

### CentOS 7 / RHEL 7 ###

systemctl restart httpd
systemctl enable httpd

### CentOS 6 / RHEL 6 ###

service httpd start
chkconfig httpd on

Configure Nagios Server
Sample configuration files have now been installed in the /usr/local/nagios/etc directory. These sample files should work fine for getting started with Nagios. You’ll need to make just one change before you proceed.

Edit the /usr/local/nagios/etc/objects/contacts.cfg config file with your favorite editor and change the email address associated with the nagiosadmin contact definition to the address you’d like to use for receiving alerts.

vi /usr/local/nagios/etc/objects/contacts.cfg

Change the Email address field to receive the notification.



define contact{
        contact_name                    nagiosadmin             ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Nagios Admin            ; Full name of user

        email                           admin@itzgeek.com       ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
        }
        
        
Install Nagios Plugins
Download Nagios Plugins to /tmp directory.

cd /tmp
wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
tar -zxvf nagios-plugins-2.2.1.tar.gz
cd /tmp/nagios-plugins-2.2.1/

Compile and install the Nagios plugins.

./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
make install

Start Nagios Server

Verify the sample Nagios configuration files.

/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

Output:

Nagios Core 4.4.3
Copyright (c) 2009-present Nagios Core Development Team and Community Contributors
Copyright (c) 1999-2009 Ethan Galstad
Last Modified: 2019-01-15
License: GPL

Website: https://www.nagios.org
Reading configuration data...
   Read main config file okay...
   Read object config files okay...

Running pre-flight check on configuration data...

Checking objects...
        Checked 8 services.
        Checked 1 hosts.
        Checked 1 host groups.
        Checked 0 service groups.
        Checked 1 contacts.
        Checked 1 contact groups.
        Checked 24 commands.
        Checked 5 time periods.
        Checked 0 host escalations.
        Checked 0 service escalations.
Checking for circular paths...
        Checked 1 hosts
        Checked 0 service dependencies
        Checked 0 host dependencies
        Checked 5 timeperiods
Checking global event handlers...
Checking obsessive compulsive processor commands...
Checking misc settings...

Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight check
If there are no errors, then start the Nagios service.


service nagios start
Start Nagios on system startup.

chkconfig nagios on
SELinux
See if SELinux is in Enforcing mode.

getenforce
Put SELinux in Permissive mode or disable it.

setenforce 0
To make this change permanent, you will have to modify /etc/selinux/config and reboot the system.

Firewall
Make sure to allow web server access through the firewall.

### FirwallD ###

firewall-cmd --permanent --add-service=http
firewall-cmd --reload


Access Nagios Web Interface
Now access the Nagios web interface using the following URL. You’ll be prompted for the username (nagiosadmin) and password you specified earlier.

http://ip-add-re-ss/nagios/





******NOW CONFIGURE THE CLIENT/REMOTE SERVER

Monitor Remote Linux Systems With Nagios

On Remote Linux System
Nagios Remote Plugin Executor (abbreviated as NRPE) plugin allows you to monitor applications and services running on remote Linux / Windows hosts. This NRPE Add-on helps Nagios to monitor local resources like CPU, Memory, Disk, Swap, etc. of the remote host.

Install NRPE Add-on & Nagios Plugins
CentOS / RHEL
NRPE Server and Nagios plugins are available in the EPEL repository for CentOS / RHEL. So, configure the EPEL repository your CentOS / RHEL system.

### CentOS 8 / RHEL 8 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

### CentOS 7 / RHEL 7 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

### CentOS 6 / RHEL 6 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
Use the following command to install NRPE Add-on and Nagios plugins.

yum install -y nrpe nagios-plugins-all

Ubuntu / Debian
Use the following command to install NRPE Add-on and Nagios plugins.

sudo apt update

sudo apt install -y nagios-nrpe-server nagios-plugins
Configure NRPE Add-on
Modify the NRPE configuration file to accept the connection from the Nagios server, Edit the /etc/nagios/nrpe.cfg file.

### CentOS / RHEL ###

vim /etc/nagios/nrpe.cfg

### Ubuntu / Debian ###

sudo nano /etc/nagios/nrpe.cfg
Add the Nagios servers IP address, separated by comma like below.

allowed_hosts=192.168.0.10

Configure Nagios Checks
The /etc/nagios/nrpe.cfg file contains the basic commands to check the attributes (CPU, Memory, Disk, etc.architecure) and services (HTTP, FTP, etc.) on remote hosts.

The path to Nagios plugins may change depends on your operating system architecture (i386 or x86_64).
CentOS / RHEL
vim /etc/nagios/nrpe.cfg

Below command lines let you monitor logged in users, system load, root filesystem usage, swap usage and the total number of the process with the help of Nagios plugins.

# COMMAND DEFINITIONS

...
...

command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib64/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
command[check_root]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /
command[check_swap]=/usr/lib64/nagios/plugins/check_swap -w 20% -c 10%
command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 150 -c 200

Ubuntu / Debian
sudo nano /etc/nagios/nrpe.cfg
Below command lines let you monitor logged in users, system load, root filesystem usage, swap usage and the total number of the process with the help of Nagios plugins.



# COMMAND DEFINITIONS

...
...

command[check_users]=/usr/lib/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
command[check_root]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /
command[check_swap]=/usr/lib/nagios/plugins/check_swap -w 20% -c 10%
command[check_total_procs]=/usr/lib/nagios/plugins/check_procs -w 150 -c 200

In the above command definition -w stands for warning and -c stands for critical.
Test Nagios Checks
For example, execute the below command in another terminal to see the check result.

Ubuntu 18.04:

/usr/lib/nagios/plugins/check_procs -w 150 -c 200

Output:

PROCS WARNING: 190 processes | procs=190;150;200;0;

Nagios plugin will count running processes and will warn you if the process count is more than 150, or it will report you critical if the process count is more than 200, and at the same time, the output will state OK if the count is below 150.

You can adjust the alert level as per your requirements.

Change warning to 200 and critical to 250 for testing purposes. Now you will see an OK message.

/usr/lib/nagios/plugins/check_procs -w 200 -c 250
Output:

PROCS OK: 189 processes | procs=189;200;250;0;
These command definitions have to be entered on a template file on the Nagios server host to enable the monitoring.

Restart the NRPE service.

### CentOS / RHEL ###

systemctl start nrpe

systemctl enable nrpe

### Ubuntu / Debian ### 

sudo systemctl restart nagios-nrpe-server
Firewall
Configure the firewall so that the Nagios server can able to reach the NRPE server running on a remote Linux host. Run these commands on a remote Linux machine.

FirewallD
firewall-cmd --permanent --add-port=5666/tcp

firewall-cmd --reload

IP Tables
iptables -I INPUT -p tcp --dport 5666 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

iptables -I OUTPUT -p tcp --sport 5666 -m conntrack --ctstate ESTABLISHED -j ACCEPT

/etc/init.d/iptables save

***********On Nagios Server
Install NRPE plugin
This NRPE plugin provides check_nrpe plugin which contacts the NRPE server on remote machines to check the services or resource.

CentOS / RHEL
Nagios NRPE plugin is available in the EPEL repository for CentOS / RHEL. So, configure the EPEL repository your CentOS / RHEL system.

### CentOS 8 / RHEL 8 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

### CentOS 7 / RHEL 7 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

### CentOS 6 / RHEL 6 ###

rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
Use the following command to install the NRPE plugin on your machine.

 yum -y install nagios-plugins-nrpe
 
Ubuntu / Debian
Use the following command to install the NRPE plugin on your machine.

sudo apt install -y nagios-nrpe-plugin

Edit Configuration
Edit the Nagios configuration file to include all .cfg files inside the /usr/local/nagios/etc/servers directory.

### CentOS / RHEL ###

vim /usr/local/nagios/etc/nagios.cfg

### Ubuntu / Debian ###

sudo nano /usr/local/nagios/etc/nagios.cfg

Add or uncomment the following line.

cfg_dir=/usr/local/nagios/etc/servers
Create a configuration directory.

### CentOS / RHEL ###

mkdir /usr/local/nagios/etc/servers

### Ubuntu / Debian ###

sudo mkdir /usr/local/nagios/etc/servers

Add Command Definition
Now it’s time to configure the Nagios server to monitor the remote client machine, and You’ll need to create a command definition in Nagios object configuration file to use the check_nrpe plugin.

Open the commands.cfg file.

CentOS / RHEL
vi /usr/local/nagios/etc/objects/commands.cfg

Add the following Nagios command definition to the file.

# .check_nrpe. command definition
define command{
command_name check_nrpe
command_line /usr/lib64/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$
}
Ubuntu / Debian
sudo nano /usr/local/nagios/etc/objects/commands.cfg
Add the following Nagios command definition to the file.

# .check_nrpe. command definition
define command{
command_name check_nrpe
command_line /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$
}
Add a Linux host to Nagios server
Create a client configuration file /usr/local/nagios/etc/servers/zabbix.johnlord.comm.cfg to define the host and service definitions of remote Linux host.

### CentOS / RHEL ###

vim /usr/local/nagios/etc/servers/zabbix.johnlord.comm.cfg

### Ubuntu / Debian ###

sudo nano /usr/local/nagios/etc/servers/zabbix.johnlord.comm.cfg
Copy the below content to the above file.

You can also use the following template and modify it according to your requirements. The following template is for monitoring logged in users, system load, disk usage (/ – partition), swap, and total process.

define host{
                           
            use                     linux-server            
            host_name               zabbix.johnlord.comm            
            alias                   zabbix.johnlord.comm            
            address                 192.168.0.47
                                    
}                                   
                                    
define hostgroup{                   
                                    
            hostgroup_name          linux-server            
            alias                   Linux Servers            
            members                 zabbix.johnlord.comm
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               zabbix.johnlord.comm            
            service_description     SWAP Uasge            
            check_command           check_nrpe!check_swap                          
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               zabbix.johnlord.comm            
            service_description     Root / Partition            
            check_command           check_nrpe!check_root                          
                                    
}                                   

define service{                     
                                    
            use                     local-service            
            host_name               zabbix.johnlord.comm            
            service_description     Current Users            
            check_command           check_nrpe!check_users                         
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               zabbix.johnlord.comm            
            service_description     Total Processes            
            check_command           check_nrpe!check_total_procs                   
                                    
}                                   
                                    
define service{                     
                                    
            use                     local-service            
            host_name               zabbix.johnlord.comm            
            service_description     Current Load            
            check_command           check_nrpe!check_load

}

Verify Nagios for any errors.

### CentOS / RHEL ###

/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

### Ubuntu / Debian ###

/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
Restart the Nagios server.

### CentOS / RHEL ###

systemctl restart nagios

### Ubuntu / Debian ###

sudo systemctl restart nagios
Check Nagios Monitoring
Go and check the Nagios web interface to view the new services we added just now.






***LINK TO AUTOMATE NAGIOS DOCUMENTATION THRU ANSIBLE FOR NRPE CLIENT***
https://www.neteye-blog.com/2018/04/how-to-deploy-nrpe-on-centos-7-with-ansible/







define service{

            use                     local-service
            host_name               nginx.johnlord.comm
            service_description     SWAP Uasge
            check_command           check_nrpe!check_swap

}   

define service{

            use                     local-service
            host_name               nginx.johnlord.comm
            service_description     Root / Partition
            check_command           check_nrpe!check_root

}

define service{

            use                     local-service
            host_name               nginx.johnlord.comm
            service_description     Current Users
            check_command           check_nrpe!check_users

}

define service{

            use                     local-service
            host_name               nginx.johnlord.comm
            service_description     Total Processes
            check_command           check_nrpe!check_total_procs

 }

define service{

            use                     local-service
            host_name               nginx.johnlord.comm
            service_description     Current Load
            check_command           check_nrpe!check_load

}                                                                                     133,1         64%
                  
