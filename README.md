# DEVOPS-TOOLING-WEBSITE-SOLUTION

**Step 1: Creating the NFS Server**
1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.
2. Based on your LVM experience from Project 6, Configure LVM on the
Server.

![partitioning xvdf](https://github.com/user-attachments/assets/385e754a-845e-4f16-a202-7b04f9592873)

![partitioning xvdg](https://github.com/user-attachments/assets/4f85e762-884b-4671-a17c-721bb35f085e)


![partitioning xvdh](https://github.com/user-attachments/assets/617f55a8-86f1-4542-b3dd-11459576aad2)

![confirming partitions lsblk](https://github.com/user-attachments/assets/c02ae45c-88c9-47cd-ad0d-9ec86ed621f8)

![sudo yum install lvm2](https://github.com/user-attachments/assets/7e3e2703-3555-4c5e-8bb4-283713e72dcf)



![sudo lvmdiskscan](https://github.com/user-attachments/assets/88071f4b-aced-450c-aede-1926799896ef)

![sudo pvcreate](https://github.com/user-attachments/assets/da94de35-c7e0-49a0-bc5a-77011184792e)


![sudo pvs](https://github.com/user-attachments/assets/8aa91f7e-3621-4c12-9be2-249b698b1767)


![sudo vgcreate nfsdata-vg](https://github.com/user-attachments/assets/ce6f8d6f-4680-47dc-b947-d7c8a1fa9005)


![sudo vgs](https://github.com/user-attachments/assets/918bdff7-36d6-46f4-9567-c3ba138c5942)


![sudo lvcreate lvs](https://github.com/user-attachments/assets/87acd2f6-9037-43d0-af4c-9620cf063b44)

![sudo vgdisplay -v](https://github.com/user-attachments/assets/3498f267-00e1-4aac-8bfa-66718d90f07f)









Instead of formatting the disks as ext4 you will have to format them
as xfs


Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs

![formatting disks as xfs](https://github.com/user-attachments/assets/df027540-a232-4f2b-9743-0ec68230093a)

Create mount points on /mnt directory for the logical volumes as
follows: Mount lv-apps on /mnt/apps - To be used by webservers


5. Mount lv-logs on /mnt/logs - To be used by webserver logs Mount lv￾opt on /mnt/opt - To be     used by Jenkins server in Project 8



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
   Make sure we set up permission that will allow our Web servers to read,
   write and execute files on NFS:


            sudo chown -R nobody: /mnt/apps
            sudo chown -R nobody: /mnt/logs
            sudo chown -R nobody: /mnt/opt
            sudo chmod -R 777 /mnt/apps
            sudo chmod -R 777 /mnt/logs
            sudo chmod -R 777 /mnt/opt
            sudo systemctl restart nfs-server.service


Configure access to NFS for clients within the same subnet (example of
Subnet CIDR - 172.31.32.0/20 ):
            sudo nano /etc/exports
            
            /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
            /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
            /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
            
            sudo exportfs -arv
            
8. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

            rpcinfo -p | grep nfs
   
Important note: In order for NFS server to be accessible from your client,
you must also open following ports: TCP 111, UDP 111, UDP 20491.On the AWS Management Console, launch and RHEL ec2 instance which would serve as the NFS server.
