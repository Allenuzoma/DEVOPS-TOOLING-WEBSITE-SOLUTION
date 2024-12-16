# DEVOPS-TOOLING-WEBSITE-SOLUTION

**Step 1: Creating the NFS Server**
1. 1. Create an EC2 instance with the Redhat OS.
2. Name the instance NFS-Server

Create three EBS volumes of 10 Gib each and attach these volumes to the ec2 instance.


Next we connect our instance using the terminal

![lsblk list block](https://github.com/user-attachments/assets/a7492248-9885-4888-b7c2-59899119cd16)


 We can see the block device currently connected using the command:


       lsblk

6. To see all the mounts and free spaces available on the server:
   
    
    
    
       df -h


   ![mounts and free spaces on the server](https://github.com/user-attachments/assets/b0903239-1bd5-4fbe-8aed-d1a1290315c1) 


7. We use the gdisk utility to create single partition on each of the three disks. 
   Starting with disk /dev/xvdf, enter the command:




    sudo gdisk /dev/xvdf

   

When asked for a command, type P for printing partition table, n for creating a new partion and W to write. Enter Yes to all and proceed with the process.


![partitioning xvdf](https://github.com/user-attachments/assets/385e754a-845e-4f16-a202-7b04f9592873)


We repeat the process for disks /dev/xvdg and /def/xvdh

![partitioning xvdg](https://github.com/user-attachments/assets/4f85e762-884b-4671-a17c-721bb35f085e)


![partitioning xvdh](https://github.com/user-attachments/assets/617f55a8-86f1-4542-b3dd-11459576aad2)


10. Use lsblk to confirm new partition on each disk:

   
        lsblk


11. Now we can see the newly created partitions for all the disks



![confirming partitions lsblk](https://github.com/user-attachments/assets/c02ae45c-88c9-47cd-ad0d-9ec86ed621f8)


12. Install the lvm2 package:



        sudo yum install lvm2



![sudo yum install lvm2](https://github.com/user-attachments/assets/7e3e2703-3555-4c5e-8bb4-283713e72dcf)

13. Check for available partition:




         sudo lvmdiskscan
    
    

![sudo lvmdiskscan](https://github.com/user-attachments/assets/88071f4b-aced-450c-aede-1926799896ef)

14. Use pvcreate utility to mark each of the three disks partitions to be used by 
    LVM:


            sudo pvcreate /dev/xvdf1
    
            sudo pvcreate /dev/xvdg1
    
            sudo pvcreate /dev/xvdh1


![sudo pvcreate](https://github.com/user-attachments/assets/da94de35-c7e0-49a0-bc5a-77011184792e)


 15. Verify the physical volume is created by entering:

    
         sudo pvs



![sudo pvs](https://github.com/user-attachments/assets/8aa91f7e-3621-4c12-9be2-249b698b1767)


 16. Use Vg create utility to add all 3 physical volumes to a volume group. Lets call the new volume: webdata-vg:

              sudo vgcreate nfsdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

![sudo vgcreate nfsdata-vg](https://github.com/user-attachments/assets/ce6f8d6f-4680-47dc-b947-d7c8a1fa9005)

 17. Verify the volume group is created by entering:
     
                 sugo vgs


![sudo vgs](https://github.com/user-attachments/assets/918bdff7-36d6-46f4-9567-c3ba138c5942)


18. Use lvcreate utility to create 3 logical volumes. apps-lv, logs-lv and opt-lv



          sudo lvcreate -n apps-lv -L 9G nfsdata-vg
    
          sudo lvcreate -n logs-lv -L 9G nfsdata-vg
    
          sudo lvcreate -n opt-lv -L 9G nfsdata-vg


    
    
19. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be
    used to store data for logs.

![sudo lvcreate lvs](https://github.com/user-attachments/assets/87acd2f6-9037-43d0-af4c-9620cf063b44)

![sudo vgdisplay -v](https://github.com/user-attachments/assets/3498f267-00e1-4aac-8bfa-66718d90f07f)








20. Instead of formatting the disks as ext4 you will have to format them
as xfs:





               sudo mkfs -t xfs /dev/nfsdata-vg/apps-lv
               sudo mkfs -t xfs /dev/nfsdata-vg/logs-lv
               sudo mkfs -t xfs /dev/nfsdata-vg/opt-lv

    
   

![formatting disks as xfs](https://github.com/user-attachments/assets/df027540-a232-4f2b-9743-0ec68230093a)

21. Create mount points on /mnt directory for the logical volumes as
    follows: Mount lv-apps on /mnt/apps - To be used by webservers
Mount lv-logs on /mnt/logs - To be used by webserver logs Mount lv-opt on /mnt/opt - To be used by Jenkins server in Project 8



            #Create mount points
            sudo mkdir -p /mnt/apps
            sudo mkdir -p /mnt/logs
            sudo mkdir -p /mnt/opt


6. Now we will mount the lvs on the mount points created above:


            #Mount the lvs on the mount points
            sudo mount /dev/nfsdata-vg/apps-lv /mnt/apps
            sudo mount /dev/nfsdata-vg/logs-lv /mnt/logs
            sudo mount /dev/nfsdata-vg/opt-lv /mnt/opt

   ![mount lvs on mount points](https://github.com/user-attachments/assets/99d965ec-3bd0-409f-b536-c718563eed6e)

   Enter sudo lsblk to see mount points
   
      ![lsblk to see mounts](https://github.com/user-attachments/assets/069ed01c-3877-47b9-b6b4-7522a164cf88)


![lsblk to see mount points after mounting](https://github.com/user-attachments/assets/d56134ab-2c98-45a9-81d9-415f83fd8f78)

   Now that we have confirmed the mounting we have to update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;   
Enter the command below to view the UUID:

            sudo blkid


   
![fstab edit](https://github.com/user-attachments/assets/a1718c12-32bb-4f79-b5c5-d9f927039a49)



Test the configuration and reload the daemon





         sudo mount -a
         sudo systemctl daemon-reload

         

4. Install NFS server, configure it to start on reboot and make sure it is up and running:



            sudo yum -y update
            sudo yum install nfs-utils -y
            sudo systemctl start nfs-server.service
            sudo systemctl enable nfs-server.service
            sudo systemctl status nfs-server.service

   
   ![sudo yum -y  update ](https://github.com/user-attachments/assets/8ef91e86-7531-453c-bde6-6aef8045013e)
   
   ![sudo yum install nfs-utils y ](https://github.com/user-attachments/assets/06ebd0f7-6443-4241-9fdc-09dc64b3fd15)

![sudo systemctl start and enable nfs-server service](https://github.com/user-attachments/assets/757ce41f-c648-48fa-930b-d7266b29629f)


6. Export the mounts for webservers' subnet cidr to connect as clients.
   
   For simplicity, you will install your all three Web Servers inside the same
   subnet, but in production set up you would probably want to separate
   each tier inside its own subnet for higher level of security.
   
   To check your subnet cidr - open your EC2 details in AWS web console and locate
   'Networking' tab and open a Subnet link:



   ![ipv4 cidr](https://github.com/user-attachments/assets/e3a62008-bc24-419d-949c-846f9537c5cf)




   Make sure we set up permission that will allow our Web servers to read,
   write and execute files on NFS:

            #Met the owner of the mount points to nobody
            sudo chown -R nobody: /mnt/apps
            sudo chown -R nobody: /mnt/logs
            sudo chown -R nobody: /mnt/opt
   
            #Modify the permissions of the mount poins to rwx for all users
            sudo chmod -R 777 /mnt/apps
            sudo chmod -R 777 /mnt/logs
            sudo chmod -R 777 /mnt/opt
            sudo systemctl restart nfs-server.service


   ![check permission status](https://github.com/user-attachments/assets/1437cced-2faf-47a9-9a13-e4a0c970c79a)

   


Configure access to NFS for clients within the same subnet (in our case the 
Subnet CIDR - 172.31.16.0/20 ):


            sudo nano /etc/exports

            
Enter the CIDR in this format:

            /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
            /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
            /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)


![etc exports](https://github.com/user-attachments/assets/96bdc1f1-eb47-43a9-9ebc-5523dcf3ea3d)


            
            sudo exportfs -arv

![sudo exportfs -arv](https://github.com/user-attachments/assets/a3b9e1df-6350-4b80-8bac-88a6608fc85e)

            
            
8. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule):



            rpcinfo -p | grep nfs

   ![check port used by nfs](https://github.com/user-attachments/assets/3ddfdb95-87ff-44a9-bae4-8fdfb5023bde)


Important note: In order for NFS server to be accessible from your client,
you must also open following ports: TCP 111, UDP 111, UDP 20491

![editing inbound rules](https://github.com/user-attachments/assets/2d2842b8-06f9-4025-8fa4-1e5326b7b82e)


**Step 2: Configure the Database Server**

Launch a new EC2 Redhat EL instance which will serve as the database server. 

Enter into the server using a terminal.




Enter the command below to update and install mysql-server:


        sudo yum update
        sudo yum install mysql-server

Verify that the service is up and running by using 

        sudo systemctl status mysqld


![systemctl status mysqld](https://github.com/user-attachments/assets/f9ef0465-f11e-4801-800f-bd2c0e210dc8)

We will restart the service and enable it so it will be running even after reboot:


      sudo systemctl restart mysqld
      sudo systemctl enable mysqld

![systemctl restart and enable mysqld](https://github.com/user-attachments/assets/0e1aeb1f-5eb2-4a9f-bf99-19b05ae3725c)



Configure remote access by editing the mysql configuration file with is found at /etc/my.cnf in Mysql installed on Amazon linux 2023. This is equivalent to the 

        sudo nano /etc/my.cnf

![bind-address](https://github.com/user-attachments/assets/df8c284a-df14-4dae-aeb7-f196fcef6794)



Enter the mysql console:



       sudo mysql

       

Create a secure user:


       ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password123@';


 Exit the console

     exit

 Ensure secure installation by typing the command:


 
    sudo mysql_secure_installation
    

 Create password validation.

 Select yes to ALL


 Sign in again with password:


     sudo mysql -p

Create a database called 'tooling' in the console;

     CREATE DATABASE tooling;

![login w password and creating db](https://github.com/user-attachments/assets/e80ea20d-6e8e-4fce-8025-d23596e33a5b)


Check to see the database created:

     SHOW DATABASES;

![show databases](https://github.com/user-attachments/assets/b6576e0a-565f-4e00-a510-c1d1b01f2987)

     
Exit the console;


Create new remote user using the command:



    CREATE USER 'webaccess'@'%' IDENTIFIED WITH mysql_native_password BY 'Password123$';
       
    #Allow the remote user access on any item with 'tooling' 
    GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'%' WITH GRANT OPTION;
  
    #Remove existing privileges
    FLUSH PRIVILEGES;

    

  ![create and granting permisiion to remote client mysql](https://github.com/user-attachments/assets/a30fffca-4b1c-48ee-bed8-8b087a0579ae)



     
     Create exit
     

To check the available users:



    SELECT user FROM mysql.user;



To delete or remove a remote user, use the syntax

  
   #DROP USER '[user]'@'[host]'; eg:

    DROP USER 'webaccess'@'%';



**Step 3 — Prepare the Web Servers**


We need to make sure that our Web Servers can serve the same content
from shared storage solutions, in our case - NFS Server and MySQL
database. You already know that one DB can be accessed
for reads and writes by multiple clients. 

For storing shared files that our Web
Servers will use - we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the
users (/var/www).
This approach will make our Web Servers stateless, which means we will be
able to add new ones or remove them whenever we need, and the integrity
of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:

- Configure NFS client (this step must be done on all three servers)
- Deploy a Tooling application to our Web Servers into a shared NFS folder
  
Configure the Web Servers to work with a single MySQL database
1. Launch a new EC2 instance with RHEL 8 Operating System to serve as the webserver

  
3. Install NFS client

   
        sudo yum update -y
        sudo yum install nfs-utils nfs4-acl-tools -y


5. Mount /var/www/ and target the NFS server's export for apps

   
        sudo mkdir /var/www
        sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps
        /var/www


   
7. Verify that NFS was mounted successfully by running df -h.

  ![mounting of NFS server to www var - ie where apache stores file](https://github.com/user-attachments/assets/c948c7e7-d2eb-4e00-9639-6896b2165c32)

  
   Make sure that the changes will persist on Web Server after reboot:

   
         sudo vi /etc/fstab

         
  Add following line


         <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

![fstab for nfs private ip add](https://github.com/user-attachments/assets/fc981c28-71ed-45e2-bda9-b7f7c257387f)

After editing the /etc/fstab, we will mount and reload:

        sudo mount -a
        sudo systemctl daemon-reload

       
8. Install Remi's repository, Apache and PHP




        sudo yum install httpd -y
   
        sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-  8.noarch.rpm
   
        sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
   
        sudo dnf module reset php
   
        sudo dnf module enable php:remi-7.4
   
        sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
   
        sudo systemctl start php-fpm
   
        sudo systemctl enable php-fpm
   
        sudo setsebool -P httpd_execmem 1

![installing apache remi etc](https://github.com/user-attachments/assets/eb93c05f-b849-4cbd-8cfb-f197d865b4d8)


   
Repeat steps 1-5 for another 2 Web Servers.



11. Verify that Apache files and directories are available on the Web
Server in /var/www and also on the NFS server in /mnt/apps.
If you see the same files - it means NFS is mounted correctly. You can try to create a
new file touch test.txt from one server and check if the same file is
accessible from other Web Servers.

Creating test file on webserver 2

![creating test file on web server 2](https://github.com/user-attachments/assets/fe4c6e56-6b11-4292-b9c6-58bf5979d901)


Confirming presence of test file on webserver 3

![confirming presence of test file on server 3](https://github.com/user-attachments/assets/f8e149af-6bef-46fa-87e6-2be4c11b8232)


Confirming presence of test file on NFS server

![confirming presence of test file on nfs server](https://github.com/user-attachments/assets/a12da80c-77ac-40d3-8a79-fac30ed10c1e)



13. Locate the log folder for Apache on the Web Server and mount it to
NFS server's export for logs. Repeat step №4 to make sure the mount
point will persist after reboot.

We need to mount the Apache log folder to the NFS server. Typically, Apache logs are stored in /var/log/httpd Before we mount, it is important to backup the log files to prevent the loss of log files.

On the Web Servers, create the folder for the backup files and use the rsync utility to copy the content into it as follows:

         #create new folder for backup
         sudo mkdir -p /var/backups/httpd_logs
         
         #copy from apache default log folder to new folder
         sudo rsync -av /var/log/httpd/ /var/backups/httpd_logs/



Then mount the NFS share:


         sudo mount -t nfs -o rw,nosuid [NFS-Server-Private-IP-Address]:/mnt/logs /var/log/httpd

         sudo mount -t nfs -o rw,nosuid 172.31.35.233:/mnt/logs /var/log/httpd


         
Also make sure the mount point persist after reboot by editing the /etc/fstab.

And add this line to /etc/fstab:

        [NFS-Server-Private-IP]:/mnt/logs /var/log/httpd nfs defaults 0 0


![fstab for mount logs](https://github.com/user-attachments/assets/38bfa5c7-e604-48d0-a82b-b57f4f62d1b0)



        
Restore the backed-up log files to the mounted directory using rsync:

        sudo rsync -av /var/backups/httpd_logs/ /var/log/httpd/

        
15. Fork the tooling source code from StegHub Github Account to your
Github account:



         git clone https://github.com/Allenuzoma/tooling.git
         sudo cp -R tooling/html/. /var/www/html/


![step 5 tooling](https://github.com/user-attachments/assets/7b7ab56e-fe39-4e61-966c-d8f6059b3313)




   ![web server 1 showing html file](https://github.com/user-attachments/assets/7a3bff66-9503-48d1-a48a-1b620b63ae86)


We can see that the html file is replicated in our web server 2


 ![web server 2 showing html file](https://github.com/user-attachments/assets/957ee2aa-e7fd-4331-9c0d-850426c228b0)

It is replicated in our NFS file server as well
![nfs server showing html file](https://github.com/user-attachments/assets/a73bb651-ff05-4b83-b9d9-eaa3adbec5d0)


16. Deploy the tooling website's code to the Webserver. Ensure that
the html folder from the repository is deployed to /var/www/html

Note 1: Do not forget to open TCP port 80 on the Web Server.


Restart Apache:

         sudo systemctl restart httpd

![sudo systemctl restart httpd](https://github.com/user-attachments/assets/c4296230-4896-4b48-b449-c85ddf4c6e17)

![selinux enforcing](https://github.com/user-attachments/assets/1c25d063-1f47-485a-873f-fdbef83e431f)


Note 2: If you encounter 403 Error - check permissions to
your /var/www/html folder and also disable SELinux using the command:

         sudo setenforce 0 
         
         
To make this change permanent - open following config file 


        sudo nano /etc/sysconfig/selinux 


Set SELINUX=disabled, then restart httpd.

![selinux disabled](https://github.com/user-attachments/assets/f250c982-109f-433b-bebe-7938c3261f87)


![systemcl status httpd](https://github.com/user-attachments/assets/7bcd2a1a-35f7-4be6-9b0a-7ec9d6c949d9)


18. Update the website's configuration to connect to the database
(in /var/www/html/functions.php file).

  On the Webserver CLI enter the command:


     sudo nano /var/www/html/functions.php


  Edit the configuration to connect the database using the syntax

     # $db = mysqli_connect('<database_server_private_ip>', '<remote_user>', '<password>', '<tooling>');

     $db = mysqli_connect('172.31.24.252', 'webaccess', 'Password123$', 'tooling');



![functions php with db priv ip](https://github.com/user-attachments/assets/8fa18a93-ab84-4525-9a5f-6919d9caaee9)


Open the mysql port 3306 on the database server before this.
 
21. On your Webserver, apply tooling-db.sql script to your database using this command.


    #mysql -h <databse-private-ip> -u <db_remote_username> -p tooling < 
      tooling-db.sql




      mysql -h 172.31.24.252 -u webaccess -p tooling < tooling-db.sql


  Enter the request password for the remote user  


 Open the mysql port 3306 on the database server before this.


![moving tooling-sql to the db](https://github.com/user-attachments/assets/793c0394-8a32-487e-b395-d8fac7115037)


21. Create in MySQL database server a new admin user with username: myuser and
password: Password123$

![creating myuser on db server](https://github.com/user-attachments/assets/2cb287f4-5130-412c-ad2d-bdd640dd53ae)

![grant privileges](https://github.com/user-attachments/assets/ab07c3b2-6ca9-4481-a4c8-7ea2f4d542d2)

22. From Webserver 1, log into the Mysql Server and the database 'tooling':




           mysql -h [database-private-ip] -u [db-username] -p [db-name]



   
   
   Login to the db server remotely






           mysql -h 172.31.24.252 -u myuser -p tooling



   ![login db server from webserver](https://github.com/user-attachments/assets/d88557f7-d158-42ab-ad8e-19d2913c46ad)

    
From the console, enter this command to create a new table 'users':


        INSERT INTO 'users' ('id', 'username', 'password', 'email', 'user_type', 'status') VALUES (1, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');

        
24. Open the website in your browser http://<Web-Server-Public-IP-Address-or Public-DNS-Name>/index.php and make sure you can login into the websute
with myuser user.

![steghub tooling login](https://github.com/user-attachments/assets/454abd39-1e84-402d-9f03-a3064abd5c17)


![admin page](https://github.com/user-attachments/assets/b147a2d9-cae3-4226-b270-7eda2ad57d77)


We can see that the website is accessible from webserver 1. This also happens when we use the other webservers 2 and 3.
We are able to login and access the same database and NFS contents as webserver 1.

 

       
