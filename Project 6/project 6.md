# WEB SOLUTION WITH WORDPRESS PROJECT 6

#### This project is written with Ubuntu 20.04 as the host machine and RHEL-8.4.0 as the remote server on AWS

Three-tier Architecture Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprises of 3 separate layers.

* Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.

* Business Layer (BL): This backend program implements business logic. Application or Webserver

* Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

Your 3-Tier Setup
A Laptop or PC to serve as a client
An EC2 Linux Server as a web server (This is where you will install WordPress)
An EC2 Linux server as a database (DB) server

###

1. **Create an AWS instance using RedHat Distribution**

The EC2 instance will serve as a Web Server, creating 3 volumes in the same AZ as the Web Server EC2, each of 10GB. We then going to create 3 volumes and attach the three volumes one by one to your Webserver EC2 instance

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/1.PNG)

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/2.PNG)

3. Open up the Linux terminal to begin configuration

4. Use `lsblk` command to inspect what block devices are attached to the server. Notice the names of your newly created devices.

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/3.PNG)

5. Use gdisk utility to create a single partition on each of the 3 disks
`sudo gdisk /dev/xvdf`

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/4.PNG)

6 Use ``lsblk` utility to view the newly configured partition on each of the 3 disks.

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/5.PNG)

7 Install lvm2 package using `sudo yum install lvm2`. Run sudo `lvmdiskscan` command to check for available partitions.
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/6.PNG)

8 Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`
 Verify that your Physical volume has been created successfully by running `sudo pvs`

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/7.PNG)

 9 Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
Verify that your VG has been created successfully by running 'sudo vgs'
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/8.PNG)

10 Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. Verify that your Logical Volume has been created successfully by running 'sudo lvs'

'sudo lvcreate -n apps-lv -L 14G webdata-vg'
'sudo lvcreate -n logs-lv -L 14G webdata-vg'

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/9.PNG)

11 Use `mkfs.ext4` to format the logical volumes with `ext4` filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

12  Create /var/www/html directory to store website files
**`sudo mkdir -p /var/www/html`**

 Create /home/recovery/logs to store backup of log data
**`sudo mkdir -p /home/recovery/logs`**

Mount /var/www/html on apps-lv logical volume
**`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`**

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
**`sudo rsync -av /var/log/. /home/recovery/logs/`**

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/10.PNG)

Run the command 'sudo blkid'
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/11.PNG)

13 Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
'sudo vi /etc/fstab'

Test the configuration and reload the daemon
```
 sudo mount -a
 sudo systemctl daemon-reload
```
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/12.PNG)

#### Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of `apps-lv` create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`.

**Install WordPress on your Web Server EC2**

Update the repository

**`sudo yum -y update`**

Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```

To install PHP and it’s depemdencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

SInstall MySQL on your DB Server EC2

```
sudo yum update
sudo yum install mysql-server
```

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

Configure DB to work with WordPress

```
sudo mysql

CREATE DATABASE wordpress;

CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';

GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';

FLUSH PRIVILEGES;

SHOW DATABASES;

exit
```


Configure WordPress to connect to remote database.


Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32


Install MySQL client and test that you can connect from your Web Server to your DB 
server by using mysql-client

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/13.PNG)

Change permissions and configuration so Apache could use WordPress:

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%206/images/14.png)

Try to access from your browser the link to your WordPress `http://<Web-Server-Public-IP-Address>/wordpress/`















