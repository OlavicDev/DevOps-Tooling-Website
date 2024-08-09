# DevOps-Tooling-Website

## Introduction to DevOps Tooling Website Solution-101
In our journey of enhancing DevOps capabilities, we previously created a WordPress-based solution ready to be utilized as a dynamic website or blog. Now, we aim to elevate our project by integrating essential DevOps tools that our team will leverage for managing, developing, testing, deploying, and monitoring various projects. This new initiative, "DevOps Tooling Website Solution-101," will empower our team with robust tools widely adopted in the industry.

### DevOps Tools Overview
Our DevOps Tooling Solution will encompass the following critical tools:

1. `Jenkins`: A powerful, open-source automation server for building CI/CD pipelines.

2. `Kubernetes`: An advanced container orchestration system to automate deployment, scaling, and management of applications.
3. `JFrog Artifactory`: A universal repository manager supporting all major packaging formats, build tools, and CI servers.
4. `Rancher`: A versatile platform for running and managing Docker and Kubernetes in production.
5. `Grafana`: An open-source analytics and interactive visualization web application for monitoring and metrics.
6. `Prometheus`: A comprehensive monitoring system with a flexible query language, efficient time series database, and modern alerting mechanisms.
7. `Kibana`: An intuitive user interface for visualizing Elasticsearch data and navigating the Elastic Stack.


### Self-Study Recommendations
To enhance your knowledge, explore the following topics:

a. Network-attached storage (NAS),  Storage Area Network (SAN), and related protocols like NFS, (s)FTP, SMB, iSCSI.

b. Block-level storage and its usage by Cloud Service providers, distinguishing it from Object storage.

c. AWS services to understand the differences between Block Storage, Object Storage, and Network File System.

## Project 7: Setup and Technologies
In this project, you will implement a tooling website solution to make DevOps tools easily accessible within our corporate infrastructure. The solution will include:

Infrastructure: Hosted on AWS

Web Server: Red Hat Enterprise Linux 8

Database Server: Ubuntu 24.04 with MySQL

Storage Server: Red Hat Enterprise Linux 8 with NFS Server

Programming Language: PHP

Code Repository: GitHub

![image](https://github.com/user-attachments/assets/b13f8152-31be-4fe5-b9de-35c67b7ad195)


Implementation Strategy

This deployment will follow a common pattern where multiple stateless web servers share a common database and access files through a Network File System (NFS). Even if the NFS server is on separate hardware, the web servers will interact with it as if it were a local file system, ensuring seamless file sharing.

### Key Considerations
Choosing the right storage solution involves answering several critical questions:

What type of data will be stored?

In what format?

How will the data be accessed and by whom?

From where and how frequently will the data be accessed?

By addressing these questions, you will be equipped to select the most suitable storage system for your solution.


## Step 1 - Prepare the NFS Server

1. **Launch an EC2 Instance with RHEL OS**

   - Begin by launching an EC2 instance running the Red Hat Enterprise Linux (RHEL) operating system.
   - Configure Logical Volume Management (LVM) on the server:
     - Format the LVM as XFS.
     - Create three Logical Volumes: `lv-opt`, `lv-apps`, and `lv-logs`.
   - Create mount points in the `/mnt` directory for the logical volumes as follows:
     - Mount `lv-apps` to `/mnt/apps` (to be utilized by web servers).
     - Mount `lv-logs` to `/mnt/logs` (to be used for storing web server logs).
     - Mount `lv-opt` to `/mnt/opt` (this will be used by the Jenkins server in the next project).
   - Create three volumes, each with a capacity of 10GB, in the same Availability Zone (AZ) as the NFS Server EC2 instance. Attach these volumes one by one to the NFS Server.
![image](https://github.com/user-attachments/assets/4d2ea5df-f45c-4321-b7b6-15b8537937ed)

2. **Initiate NFS Server Configuration**

   - Open the Linux terminal to begin configuration:
   ```
   ssh -i "yourkey.pem" ec2-user@<NFS-Server-IP>
   ```
   - Use the `lsblk` command to inspect the block devices attached to the server. In Linux, all devices reside in the `/dev/` directory. Use `ls /dev/` to verify that all three newly created devices are present. Their names are likely to be `xvdf`, `xvdg`, and `xvdh`.
   ![image](https://github.com/user-attachments/assets/8c9647c0-6a6a-447b-82cc-815573f5152d)

   - Utilize the `gdisk` utility to create a single partition on each of the three disks:
   ```
   sudo gdisk /dev/xvdf
   sudo gdisk /dev/xvdg
   sudo gdisk /dev/xvdh
   ```
   ![image](https://github.com/user-attachments/assets/dcaa2559-4ca6-4d63-a617-d3b9d7f92f73)

   - After partitioning, view the newly configured partitions on each of the three disks using the `lsblk` command.
    ![image](https://github.com/user-attachments/assets/7f2d5d6a-c6ef-4811-9c4b-de78d25a3009)

   - Install the LVM package:
   ```
   sudo yum install lvm2 -y
   ```
   ![image](https://github.com/user-attachments/assets/8320eab8-f1ac-4071-a09b-13b7420dce9e)

   - Use the `pvcreate` utility to designate each of the three disks as Physical Volumes (PVs) for use by LVM. Confirm that the volumes have been successfully created:
   ```
   sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
   sudo pvs
   ```
   - Utilize the `vgcreate` utility to add all three PVs to a Volume Group (VG) named `webdata-vg`. Verify that the VG has been successfully created:
   ```
   sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
   sudo vgs
   ```
   ![image](https://github.com/user-attachments/assets/ed03ce0e-8618-4717-b458-653d5cabe006)

   - Create three logical volumes named `lv-apps`, `lv-logs`, and `lv-opt` using the `lvcreate` utility. Verify the creation of the logical volumes:
   ```
   sudo lvcreate -n lv-apps -L 9G webdata-vg
   sudo lvcreate -n lv-logs -L 9G webdata-vg
   sudo lvcreate -n lv-opt -L 9G webdata-vg
   sudo lvs
   ```
   - Verify the entire setup:
   ```
   sudo vgdisplay -v   # view complete setup, VG, PV, and LV
   lsblk
   ```
   ![image](https://github.com/user-attachments/assets/cdf85328-6464-4e8b-9647-4a0bf71ec6cc)

   - Format the logical volumes using `mkfs -t xfs` instead of the ext4 filesystem:
   ```
   sudo mkfs -t xfs /dev/webdata-vg/lv-apps
   sudo mkfs -t xfs /dev/webdata-vg/lv-logs
   sudo mkfs -t xfs /dev/webdata-vg/lv-opt
   ```
   ![image](https://github.com/user-attachments/assets/38fdba59-bbb8-4f48-8784-f240acfafb4a)

   - Create mount points in the `/mnt` directory:
   ```
   sudo mkdir /mnt/apps
   sudo mkdir /mnt/logs
   sudo mkdir /mnt/opt
   sudo mount /dev/webdata-vg/lv-apps /mnt/apps
   sudo mount /dev/webdata-vg/lv-logs /mnt/logs
   sudo mount /dev/webdata-vg/lv-opt /mnt/opt
   ```

3. **Install and Configure the NFS Server**

   - Install the NFS Server and configure it to start on reboot. Ensure that it is up and running:
   ```
   sudo yum update -y
   sudo yum install nfs-utils -y
   sudo systemctl start nfs-server.service
   sudo systemctl enable nfs-server.service
   sudo systemctl status nfs-server.service
   ```
   ![image](https://github.com/user-attachments/assets/c592de50-e09e-4a0f-af1e-93eadf11f4f8)

   - Export the mounts to allow Webservers' subnet CIDR (IPv4 CIDR) to connect as clients. For simplicity, all three Web Servers are installed in the same subnet. However, in a production setup, each tier should be separated within its own subnet for higher security.
   - Set up permissions to allow the Web Servers to read, write, and execute files on NFS:
   ```
   sudo chown -R nobody: /mnt/apps
   sudo chown -R nobody: /mnt/logs
   sudo chown -R nobody: /mnt/opt
   sudo chmod -R 777 /mnt/apps
   sudo chmod -R 777 /mnt/logs
   sudo chmod -R 777 /mnt/opt
   sudo systemctl restart nfs-server.service
   ```
   - Configure access to NFS for clients within the same subnet (example Subnet CIDR: 172.31.32.0/20):
   ```
   sudo vi /etc/exports
   ```
   - Add the following lines to `/etc/exports`:
   ```
   /mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
   /mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
   /mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
   ```
   - Apply the exports using `exportfs`:
   ```
   sudo exportfs -arv
   ```
   - Identify the port used by NFS and open it using the security group (add a new inbound rule):
   ```
   rpcinfo -p | grep nfs
   ```
   ![image](https://github.com/user-attachments/assets/959fd9e0-f8b1-4f65-aa02-e84f323eab6f)

   - Note: For the NFS Server to be accessible from the client, the following ports must be opened: TCP 111, UDP 111, UDP 2049, and NFS 2049. Set the Web Server subnet CIDR as the source.

---

### Step 2 - Configure the Database Server

1. **Launch an Ubuntu EC2 Instance for the Database Server**

   - Launch a new EC2 instance running Ubuntu, which will serve as the Database (DB) Server.
   - Access the instance to begin configuration:
   ```
   ssh -i "yourkey.pem" ubuntu@<DB-Server-IP>
   ```
   - Update and upgrade the Ubuntu instance:
   ```
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install MySQL Server**

   - Install the MySQL Server package:
   ```
   sudo apt install mysql-server -y
   ```
   - Run the MySQL secure installation script to secure the MySQL installation:
   ```
   sudo mysql_secure_installation
   ```
   ![image](https://github.com/user-attachments/assets/c18c4230-c06e-4198-ace2-55ddb6824af2)
   ![image](https://github.com/user-attachments/assets/0aba51d0-2f5c-4526-99b3-4f7a4e6d392e)



4. **Create the Database and Configure User Access**

   - Create a database named `tooling` and a database user named `webaccess`:
   ```
   sudo mysql
   CREATE DATABASE tooling;
   CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
   GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.0.0/20' WITH GRANT OPTION;
   FLUSH PRIVILEGES;
   exit;
   ```
   ![image](https://github.com/user-attachments/assets/4665734b-501c-4d6f-b6a4-b2055d850d00)

   - Configure MySQL to allow remote connections by setting the bind address and then restart MySQL:
   ```
   sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   - Restart the MySQL service:
   ```
   sudo systemctl restart mysql
   sudo systemctl status mysql
   ```
   ![image](https://github.com/user-attachments/assets/2394a624-7f62-424d-93cc-3c7d6e47e140)

   - Open MySQL port 3306 on the DB Server EC2 instance. Access to the DB Server is restricted to the subnet CIDR configured as the source.

---

### Step 3 - Prepare the Web Servers

To ensure that the Web Servers can serve the same content from a shared storage solution (NFS) and a single MySQL database, the following steps were undertaken:

1. **Launch EC2 Instances for Web Servers (RHEL OS)**

   - Launch a new EC2 instance with the RHEL operating system for each Web

 Server.

2. **Install the NFS Client on Each Web Server**

   - Install the NFS client package and mount the NFS server export directory to `/var/www/`:
   ```
   sudo yum install nfs-utils nfs4-acl-tools -y
   sudo mkdir /var/www
   sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP>:/mnt/apps /var/www
   ```
   - Ensure that the mount persists by adding it to `/etc/fstab`:
   ```
   <NFS-Server-Private-IP>:/mnt/apps /var/www nfs defaults 0 0
   ```

3. **Install Apache, PHP, and Required Packages**

   - Install Apache and PHP packages, including the necessary extensions:
   ```
   sudo yum install httpd -y
   ```
   ![image](https://github.com/user-attachments/assets/3e6a8ffd-4be8-4746-a883-26d0b7d16cec)

   ```
   sudo systemctl enable httpd
   sudo systemctl start httpd
   ```
   
   ```
   sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
   ```
   ![image](https://github.com/user-attachments/assets/15729276-bcf4-4194-8f5d-2e64e6102359)

   ```
   sudo dnf module reset php
   sudo dnf module enable php:remi-8.2
   ```
   ![image](https://github.com/user-attachments/assets/52b9ae8d-e5dd-40c9-83f6-80a1f76a1012)

   ```
   sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   sudo setsebool -P httpd_execmem 1
   ```
   ![image](https://github.com/user-attachments/assets/b957bf94-0309-4375-b075-e90b470345ab)

**REPEAT THE PROCESS FOR THE REMAINING TWO WEB SERVERS**

5. **Deploy the Tooling Website**

   - Fork the repository and clone it to the Web Server. Deploy the code to `/var/www/html`:
  
   ![image](https://github.com/user-attachments/assets/22c630ac-1dc9-4053-8b3c-acb56f4c4c87)

   ```
   git clone https://github.com/<your-username>/tooling.git
   sudo cp -R tooling/html/. /var/www/html/
   sudo vi /var/www/html/functions.php
   ```
   - Update the MySQL database connection settings in `functions.php`:
   ```
   <?php
   define('DB_SERVER', ' <DB-Server-Private-IP> ');
   define('DB_USERNAME', 'webaccess');
   define('DB_PASSWORD', 'Admin123$');
   define('DB_DATABASE', 'tooling');
   $db = mysqli_connect(DB_SERVER,DB_USERNAME,DB_PASSWORD,DB_DATABASE);
   ?>
   ```
   - Create an inbound rule in the Web Server Security Group allowing HTTP and HTTPS traffic from any IP address, and another rule allowing MySQL connection from Web Server Subnet CIDR.

6. **Final Steps and Verification**

   - Test the Tooling website by accessing it from a browser using the Web Server's public IP address:
   ```
   http://<Web-Server-Public-IP-1-or-2>/index.php
   ```
   ![image](https://github.com/user-attachments/assets/5f9b5b36-b158-4dd6-beeb-fd6ec489694e)
   ![image](https://github.com/user-attachments/assets/e06349b3-a394-4150-aba4-15039eaffe20)


   - Verify that Apache is running properly on both Web Servers, and that the application can connect to the MySQL database. Check logs to ensure everything is functioning as expected:
   ```
   sudo tail -f /var/log/httpd/error_log
   ```


## Project Overview

The DevOps Tooling Website Solution project aims to establish a comprehensive DevOps environment that integrates various tools and components necessary for managing, developing, testing, deploying, and monitoring applications. The project involves setting up infrastructure on AWS, configuring servers for web hosting, database management, and storage, and integrating these components with DevOps tools to create a robust, scalable, and efficient system for continuous integration and deployment.

### Importance of the Project

1. **Centralized Infrastructure Management:**
   - By deploying the infrastructure on AWS and using EC2 instances, the project leverages cloud scalability, reliability, and security. AWS provides a flexible environment where resources can be managed effectively, ensuring high availability and performance.

2. **Standardized Environment:**
   - The use of Red Hat Enterprise Linux (RHEL) and Ubuntu ensures a consistent operating environment across the infrastructure. This standardization reduces the chances of environment-specific bugs and streamlines management processes.

3. **Efficient Storage Solutions:**
   - The Network File System (NFS) server on RHEL allows multiple web servers to share the same content, ensuring consistency and reducing redundancy. The logical volumes (LVM) and formatted disks on NFS provide organized and efficient storage for different server needs, such as applications, logs, and Jenkins.

4. **Database Centralization:**
   - A centralized MySQL database on Ubuntu allows the web servers to connect to a single database instance, ensuring data consistency and simplifying database management. By restricting access to the web servers' subnet, the project enhances security.

5. **Seamless Integration and Automation:**
   - The project uses PHP as the programming language, GitHub as the code repository, and integrates various DevOps tools. This setup facilitates continuous integration and continuous deployment (CI/CD), automating the software delivery process and reducing manual errors.

6. **Scalability and Flexibility:**
   - The architecture is designed to scale easily. Additional web servers can be added to the existing setup without disrupting the service. The use of NFS and MySQL ensures that all servers can access the same data and resources, promoting horizontal scaling.

7. **Security and Compliance:**
   - By using security groups and configuring the appropriate inbound rules, the project ensures that the infrastructure is secure from unauthorized access. The use of RHEL and Ubuntu, known for their enterprise-level security features, adds an additional layer of protection.

8. **Comprehensive Documentation:**
   - The detailed step-by-step documentation provides a clear guide for setting up each component, making it easier for teams to replicate, troubleshoot, and maintain the infrastructure.

### Conclusion

This project is a significant step toward establishing a robust DevOps environment. It addresses the essential aspects of infrastructure management, storage, database integration, and application deployment. By leveraging AWS and integrating various open-source tools, the project ensures scalability, security, and efficiency, making it an ideal solution for modern software development and operations.
