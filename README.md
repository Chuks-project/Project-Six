# PROJECT SIX: WEB SOLUTION WITH WORDPRESS

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

## Project 6 consists of two parts:

- Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks,             partitions and volumes in Linux.

- Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

- Also in this project, we shall be dealing  with or rather be implementing a Three-tier Architecture which is a client-server software architecture pattern that comprises of 3 separate layers namely:

    - Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
      
    - Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
    
    - Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or         NFS Server
    
    
    # Step 0 - provisioning of Ec2 instances
Again refer to step 0 of project-one if you have forgotten how to spin an instance on AWS 

Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

Open up the Linux terminal to begin configuration

Use lsblk command to inspect what block devices are attached to the server and df -h command to see all mounts and free space on your server

 ![Block devices](https://user-images.githubusercontent.com/65022146/195098725-c4db8ff5-2564-487b-9e45-f25c1c405652.png)
 
Use gdisk utility to create a single partition on each of the 3 disks by running these commands:

    `sudo gdisk /dev/xvdf`
  
    `sudo gdisk /dev/xvdh`
  
    `sudo gdisk /dev/xvdg`
  
- Notice that All devices in Linux reside in /dev/ directory

- Use lsblk utility to view the newly configured partition on each of the 3 disks as seen in the screenshot below:


![Configured partition](https://user-images.githubusercontent.com/65022146/195101620-7b1be0e2-3973-43b1-8e9b-2641a9c0b26e.png)

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

    `sudo pvcreate /dev/xvdf1`
  
    `sudo pvcreate /dev/xvdg1`
  
    `sudo pvcreate /dev/xvdh1
  
 - Verify that your Physical volume has been created successfully by running sudo pvs
 
 
    ![pvcerate and sudo pvs](https://user-images.githubusercontent.com/65022146/195104982-56882a9f-fecb-43fb-a39a-05691fe0735e.png)
    

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

   `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
   
- Verify that your VG has been created successfully by running 
   
      `sudo vgs`

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used     to store data for the Website while, logs-lv will be used to store data for logs.
  
  `sudo lvcreate -n apps-lv -L 14G webdata-vg`
  
  `sudo lvcreate -n logs-lv -L 14G webdata-vg`
  
 - Verify that your Logical Volume has been created successfully by running
 
     `sudo lvs`
     
 - Verify the entire setup byrunning
 
     `sudo vgdisplay -v`
  
  - Then run
    
    `lsblk`
    
    
  ![volume group](https://user-images.githubusercontent.com/65022146/195111984-2043e5bb-265f-4a2a-bd15-1c41eb0486e6.png)
  
  
- Use mkfs.ext4 to format the logical volumes with ext4 filesystem

    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

- Create /var/www/html directory to store website files

   `sudo mkdir -p /var/www/html`

- Create /home/recovery/logs to store backup of log data

   `sudo mkdir -p /home/recovery/logs`

- Mount /var/www/html on apps-lv logical volume

   `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

   `sudo rsync -av /var/log/. /home/recovery/logs/`

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)

   `sudo mount /dev/webdata-vg/logs-lv /var/log`

- Restore log files back into /var/log directory

    `sudo rsync -av /home/recovery/logs/. /var/log`

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.

- The UUID of the device will be used to update the /etc/fstab file;

- Update /etc/fstab using the UUID generated for each partion respectively see example below:

   
   ![UUID](https://user-images.githubusercontent.com/65022146/195115713-04828c04-f09b-4300-bae4-ef3a4969b2a7.png)
   
- Test the configuration and reload the daemon by running:

    `sudo mount -a`
    `sudo systemctl daemon-reload`
    
- Verify your setup by running df -h, output must look like this:

   ![df -h  to verify setup after fstab in db](https://user-images.githubusercontent.com/65022146/195117228-c9835154-94e6-46c7-b735-5423a22f50df.png)

## Step 2 — Prepare the Database Server

- Repeat the disk partition process on DB server. However mount db-lv on /db/ directory instead

## Step 3 — Install WordPress on your Web Server EC2
Update the repository

`sudo yum -y update`

- Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

## Start Apache

`sudo systemctl enable httpd`
`sudo systemctl start httpd`

## To install PHP and it’s depemdencies
`
     sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
     sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo yum module list php
     sudo yum module reset php
     sudo yum module enable php:remi-7.4
     sudo yum install php php-opcache php-gd php-curl php-mysqlnd
     sudo systemctl start php-fpm
     sudo systemctl enable php-fpm
     setsebool -P httpd_execmem 1    
`
    
 ## Restart Apache
   
 `sudo systemctl restart httpd`
 
 ## Download wordpress and copy wordpress to var/www/html
 `
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
`
## Configure SELinux Policies
`
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
`

## Install MySQL on your DB Server EC2
`
sudo yum update
sudo yum install mysql-server
 `
 
## Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
`
sudo systemctl restart mysqld
sudo systemctl enable mysqld
`

## Configure DB to work with WordPress
`
Sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
`

## Configure WordPress to connect to remote database.

- Open TCP port 3306 on DB server to allow connection to the database. Also set bind-address to 0.0.0.0

- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

 `sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

- If the connection is successful, you will see an output like the one below:

![Connected to the Database from the Webserver](https://user-images.githubusercontent.com/65022146/195126358-dab87926-cb4f-47c3-b23e-db2c06aed852.png)

## Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases

- If successful, an output like the one below will appear

![Databases created](https://user-images.githubusercontent.com/65022146/195129647-61e83747-7764-4115-a8f2-ce46bcbab8f0.png)

