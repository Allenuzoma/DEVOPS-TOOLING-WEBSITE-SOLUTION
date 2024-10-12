# DEVOPS-TOOLING-WEBSITE-SOLUTION

**Step 1: Creating the NFS Server**
1. 1. Create an EC2 instance with the Redhat OS.
2. Name the instance Web-Server

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
    
19.NOTE: apps-lv will be used to store data for the Website while, logs-lv will be
    used to store data for logs.

![sudo lvcreate lvs](https://github.com/user-attachments/assets/87acd2f6-9037-43d0-af4c-9620cf063b44)

![sudo vgdisplay -v](https://github.com/user-attachments/assets/3498f267-00e1-4aac-8bfa-66718d90f07f)









Instead of formatting the disks as ext4 you will have to format them
as xfs


![formatting disks as xfs](https://github.com/user-attachments/assets/df027540-a232-4f2b-9743-0ec68230093a)

Create mount points on /mnt directory for the logical volumes as
follows: Mount lv-apps on /mnt/apps - To be used by webservers


5. Mount lv-logs on /mnt/logs - To be used by webserver logs Mount lv￾opt on /mnt/opt - To be used by Jenkins server in Project 8



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


