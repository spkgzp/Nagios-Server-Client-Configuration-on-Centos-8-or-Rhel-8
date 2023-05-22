## Nagios Server & Client Configuration on Centos-8 or Rhel-8
### 1. Setup of Nagios-server  
### All steps for nagios-server

- Set hostname of our nagios server.
```bash
  # hostnamectl set-hostname nagios-server.satyadomain.local
```
- Edit "/etc/hosts" file  and save ip and domain-name.
```bash
  192.168.1.100 nagios-server.satyadomain.local
  192.168.1.200 nagios-client.satyadomain.local
```
- Check SElinux "getenforce" or "sestatus" commands if inforcing mode plese exec permissive mode using below commands.
```bash
# setenforce 0
or
# sed -i 's/SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config && setenforce 0
```
- Check "getenforce" command permissive or not.
- Update system using below command.
```bash
# dnf update -y
````
- Now install all packageses for nagios-servers.
```bash
# dnf install @php @perl @httpd wget unzip glibc automake glibc-common gettext autoconf php php-cli gcc gd gd-devel net-snmp openssl-devel unzip net-snmp postfix net-snmp-utils -y
# dnf groupinstall "Development Tools" -y
```
- Start httpd service and set on boot time.
```bash
# systemctl enable --now httpd php-fpm
```
- Now check httpd status.
```bash
# systemctl status httpd
```
- Now download nagios-server(4.4.6) and exrtact the file using below commands.
```bash
# export VER="4.4.6"
# curl -SL https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-$VER/nagios-$VER.tar.gz | tar -xzf -
# cd nagios-$VER
# ls
```
- Run the "configure" file and using below commands.
```bash
# ./configure
# make all
# make install-groups-users
# usermod -a -G nagios apache
# make install 
# make install-daemoninit
# make install-commandmode
# make install-config
# make install-webconf
# make install-exfoliation
# make install-classicui
# htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
pass: redhat
sucessfully add.
# systemctl restart httpd
# cd
```
- Now download and install nagios plugins.
```bash
# VER="2.3.3"
# curl -SL https://github.com/nagios-plugins/nagios-plugins/releases/download/release-$VER/nagios-plugins-$VER.tar.gz | tar -xzf -
# cd nagios-plugins-2.3.3/
# ./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make
# make install
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
- Start and enable nagios service and check status.
```bash
# systemctl enable --now nagios
# systemctl status nagios
```
- Now enable firewall for http and https.
```bash
# firewall-cmd --permanent --add-service={http,https}
# firewall-cmd --reload
```
### Now open Browser 
- Check nagios dashboard 
```bash
 http://localhost/nagios
```
- Download NRPE{Nagios Remote Plugins Executer} and install.
```bash
# wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz
# tar zxf nrpe-*.tar.gz
# cd nrpe-4.0.3/
# ./configure --enable-command-args
# make all 
# make install-groups-users
# make install 
# make install-config
```
- Update services file and set boot time.
```bash
# sh -c "echo >> /etc/services"
# sh -c "sudo echo '# Nagios services' >> /etc/services"
# sh -c "sudo echo 'nrpe 5666/tcp' >> /etc/services"
# make install-init
```
- Now enable nrpe services.
```bash
# systemctl enable nrpe
```
- Now open NRPE congiguration file, edit and save "nrpe.cfg" File.
```bash
# vim /usr/local/nagios/etc/nrpe.cfg
Edit below line:
106 allowed_hosts=127.0.0.1,<Client Machine ip>
122 dont_blame_nrpe=1

Uncumment bellow line:
313 ### MISC SYSTEM METRICS ###
314 command[check_users]=/usr/local/nagios/libexec/check_users $ARG1$
315 command[check_load]=/usr/local/nagios/libexec/check_load $ARG1$
316 command[check_disk]=/usr/local/nagios/libexec/check_disk $ARG1$
317 command[check_swap]=/usr/local/nagios/libexec/check_swap $ARG1$
318 command[check_cpu_stats]=/usr/local/nagios/libexec/check_cpu_stats.sh $ARG1$
319 command[check_mem]=/usr/local/nagios/libexec/custom_check_mem -n $ARG1$
320 
321 ### GENERIC SERVICES ###
322 command[check_init_service]=sudo /usr/local/nagios/libexec/check_init_service $ARG1$
323 command[check_services]=/usr/local/nagios/libexec/check_services -p $ARG1$
324 
325 ### SYSTEM UPDATES ###
326 command[check_yum]=/usr/local/nagios/libexec/check_yum
327 command[check_apt]=/usr/local/nagios/libexec/check_apt
328 
329 ### PROCESSES ###
330 command[check_all_procs]=/usr/local/nagios/libexec/custom_check_procs
331 command[check_procs]=/usr/local/nagios/libexec/check_procs $ARG1$
332 
333 ### OPEN FILES ###
334 command[check_open_files]=/usr/local/nagios/libexec/check_open_files.pl $ARG1$
335 
336 ### NETWORK CONNECTIONS ###
337 command[check_netstat]=/usr/local/nagios/libexec/check_netstat.pl -p $ARG1$ $ARG2$
Save the file.
```
- Add port of nrpe using firewall command.
```bash
# cd
# firewall-cmd --permanent --add-port=5666/tcp
# firewall-cmd --reload
# firewall-cmd --list-all
```
- Now Start NRPE service.
```bash
# systemctl start nrpe
```
- Create a Directory and change the permissions.
```bash
# mkdir /usr/local/nagios/etc/servers
# chown -R nagios:nagios /usr/local/nagios/etc/servers
# chmod g+w /usr/local/nagios/etc/servers
```
- Edit Nagios Server Conf File (nagios.cfg).
```bash
# vim /usr/local/nagios/etc/nagios.cfg
Uncumment 51 line:
51 cfg_dir=/usr/local/nagios/etc/servers
```
- Now add some line in commands.cfg File and Modules Define For Monitoring.
```bash
# vim /usr/local/nagios/etc/objects/commands.cfg

* Define in end below line *

define command {
 command_name check_nrpe
 command_line $USER$/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$ -a $ARG2$
}
```
- New Create File For New services.
 ```bash
 # vim /usr/local/nagios/etc/servers/yourhost.cfg

define host {
 use linux-server
 host_name nagios-client.satyadomain.local
 alias My client server
 address 192.168.122.188
 max_check_attempts 5
 check_period 24x7
 notification_interval 30
 notification_period 24x7
}

###############################################################################
#
#CLIENT SERVICE DEFINITIONS
#
###############################################################################

# Define a service to "ping" the local machine

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}



# Define a service to check the disk space of the root partition
# on the local machine.  Warning if < 20% free, critical if
# < 10% free space on partition.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     Root Partition
    check_command           check_local_disk!20%!10%!/
}



# Define a service to check the number of currently logged in
# users on the local machine.  Warning if > 20 users, critical
# if > 50 users.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     Current Users
    check_command           check_local_users!20!50
}



# Define a service to check the number of currently running procs
# on the local machine.  Warning if > 250 processes, critical if
# > 400 processes.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     Total Processes
    check_command           check_local_procs!250!400!RSZDT
}



# Define a service to check the load on the local machine.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     Current Load
    check_command           check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
}



# Define a service to check the swap usage the local machine.
# Critical if less than 10% of swap is free, warning if less than 20% is free

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     Swap Usage
    check_command           check_local_swap!20%!10%
}



# Define a service to check SSH on the local machine.
# Disable notifications for this service by default, as not all users may have SSH enabled.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     SSH
    check_command           check_ssh
    notifications_enabled   0
}



# Define a service to check HTTP on the local machine.
# Disable notifications for this service by default, as not all users may have HTTP enabled.

define service {

    use                     local-service           ; Name of service template to use
    host_name               nagios-client.satyadomain.local
    service_description     HTTP
    check_command           check_http
    notifications_enabled   0
}
```
- Now Restart Nagios service.
```bash
# systemctl restart nagios
```

### 2. Setup of Nagios-client  
### All steps for nagios-client

- Set hostname of our nagios-client
```bash
  # hostnamectl set-hostname nagios-client.satyadomain.local
```
- Edit "/etc/hosts" file  and save ip and domain-name.
```bash
  192.168.1.100 nagios-server.satyadomain.local
  192.168.1.200 nagios-client.satyadomain.local 
```
- Check SElinux "getenforce" or "sestatus" commands if inforcing mode plese exec permissive mode using below commands.
```bash
# setenforce 0
or
# sed -i 's/SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config && setenforce 0
```
- Check "getenforce" command permissive or not.
- Update system using below command.
```bash
# dnf update -y
````
- Now install all packageses for nagios-Client.
```bash
# dnf install -y gcc glibc glibc-common openssl openssl-devel perl wget
# cd /usr/src/
```
- Download NRPE{Nagios Remote Plugins Executer} and install.
```bash
# wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz
# tar zxf nrpe-*.tar.gz
# cd nrpe-4.0.3/
# ./configure --enable-command-args
# make all 
# make install-groups-users
# make install 
# make install-config
```
- Update services file and set boot time.
```bash
# sh -c "echo >> /etc/services"
# sh -c "sudo echo '# Nagios services' >> /etc/services"
# sh -c "sudo echo 'nrpe 5666/tcp' >> /etc/services"
# make install-init
```
- Now enable nrpe services.
```bash
# systemctl enable nrpe
```
- Now open NRPE congiguration file, edit and save "nrpe.cfg" File.
```bash
# vim /usr/local/nagios/etc/nrpe.cfg
Edit below line:
106 allowed_hosts=127.0.0.1,<Server Machine ip>
122 dont_blame_nrpe=1

Uncumment bellow line:
313 ### MISC SYSTEM METRICS ###
314 command[check_users]=/usr/local/nagios/libexec/check_users $ARG1$
315 command[check_load]=/usr/local/nagios/libexec/check_load $ARG1$
316 command[check_disk]=/usr/local/nagios/libexec/check_disk $ARG1$
317 command[check_swap]=/usr/local/nagios/libexec/check_swap $ARG1$
318 command[check_cpu_stats]=/usr/local/nagios/libexec/check_cpu_stats.sh $ARG1$
319 command[check_mem]=/usr/local/nagios/libexec/custom_check_mem -n $ARG1$
320 
321 ### GENERIC SERVICES ###
322 command[check_init_service]=sudo /usr/local/nagios/libexec/check_init_service $ARG1$
323 command[check_services]=/usr/local/nagios/libexec/check_services -p $ARG1$
324 
325 ### SYSTEM UPDATES ###
326 command[check_yum]=/usr/local/nagios/libexec/check_yum
327 command[check_apt]=/usr/local/nagios/libexec/check_apt
328 
329 ### PROCESSES ###
330 command[check_all_procs]=/usr/local/nagios/libexec/custom_check_procs
331 command[check_procs]=/usr/local/nagios/libexec/check_procs $ARG1$
332 
333 ### OPEN FILES ###
334 command[check_open_files]=/usr/local/nagios/libexec/check_open_files.pl $ARG1$
335 
336 ### NETWORK CONNECTIONS ###
337 command[check_netstat]=/usr/local/nagios/libexec/check_netstat.pl -p $ARG1$ $ARG2$
Save the file.
```
- Add port of nrpe using firewall command.
```bash
# cd
# firewall-cmd --permanent --add-port=5666/tcp
# firewall-cmd --reload
# firewall-cmd --list-all
```
- Now Start NRPE service.
```bash
# systemctl start nrpe
```
- #### Now open browser and search <http://localhost/nagios> and show dashboard.
![image](https://user-images.githubusercontent.com/113234348/211195309-27d28409-c5ff-4234-b990-20a5ed1e6279.png)
![image](https://user-images.githubusercontent.com/113234348/211195348-65231cd8-efd0-4b80-815d-e8ccbe2a88bf.png)


