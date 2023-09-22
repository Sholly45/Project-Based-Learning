# **DEVOPS TOOLING WEBSITE SOLUTION**

In this project, you will implement a solution that consists of the following components:

* Infrastructure: AWS

* Webserver Linux: Red Hat Enterprise Linux 8

* Database Server: Ubuntu 20.04 + MySQL

* Storage Server: Red Hat Enterprise Linux 8 + NFS Server

* Programming Language: PHP

* Code Repository: Github

For server, we will make use of the RHEL 8 Server on AWS

(image1)

## STEP 1 – PREPARE NFS SERVER

1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.

2. Based on your LVM experience from Project 6, Configure LVM on the Server.

*  Create 3 volumes for the NFS server

* using the gdisk utility to create partitions 

* Once the partition is created install lvm2 using `sudo yum install lvm2` then carry out the other process as in project 6.



* Instead of formating the disks as `ext4` you will have to format them as `xfs`

```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```


* Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs

3. Create mount points on /mnt directory for the logical volumes as follow:

    Mount lv-apps on /mnt/apps – To be used by webservers

    Mount lv-logs on /mnt/logs – To be used by webserver logs

    Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

```
sudo mkdir /mnt/apps

sudo mkdir /mnt/logs

sudo mkdir /mnt/opt

sudo mount /dev/webdata-vg/lv-apps /mnt/apps

sudo mount /dev/webdata-vg/lv-logs /mnt/logs

sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
* Once mount is completed run **`sudo blkid`** to get the UUID of the mount part, open and paste the UUID in the fstab file.

**`sudo vi /etc/fstab`**

```
sudo mount -a 
sudo systemctl daemon-reload
```
(image2)

## STEP 2 –  CONFIGURE THE DATABASE SERVER

1. Install MySQL server

**`sudo apt install mysql`**

2. Create a database and name it `tooling`
```
sudo mysql
create database tooling;
```

3. Create a database user and name it `webaccess`

**`create user 'webaccess'@'172.31.80.0/20' identified by 'password';`**

4. Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

**`grant all privileges on tooling.* to 'webaccess'@'172.31.80.0/20';`**

(image 3)

## STEP 3 –  CONFIGURE THE NFS SERVER UTILITIES


1. Install NFS server, configure it to start on reboot and make sure it is u and running

```
sudo yum -y update

sudo yum install nfs-utils -y

sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service
```

2. Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.
To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

Install Remi’s repository, Apache and PHP
```
sudo yum install httpd -y

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo yum install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo yum module  list php -y

sudo yum module reset php -y

sudo yum module enable php:remi-7.4 -y

sudo yum install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

sudo systemctl restart httpd

```

### **Repeat steps 1-5 for another 2 Web Servers.**

3. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

4. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step 4 under the 'prepare web servers' to make sure the mount point will persist after reboot.

`sudo vi /etc/fstab`

5. Fork the tooling source code from Darey.io Github Account to your Github account.


6. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to `/var/www/html`


**Note 1: Do not forget to open TCP port 80 on the Web Server.**

**Note 2: If you encounter 403 Error – check permissions to your `/var/www/html` folder and also disable SELinux `sudo setenforce 0`**

To make this change permanent – open following config file `sudo vi /etc/sysconfig/selinux` and set `SELINUX=disabled` then restart httpd.

```
sudo systemctl restart httpd
sudo systemctl status httpd
```


