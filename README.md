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


### Step 1: Prepare the NFS Server (RHEL 9)

1. **Launch an EC2 instance with RHEL 9**.
   - Create an EC2 instance with RHEL 9 as the operating system.
   - Ensure that the instance is in the same VPC as your web and database servers for network connectivity.

2. **Configure Logical Volume Management (LVM)**.
   - Attach three new EBS volumes (each 10GB) to the instance.
   - Connect to the instance via SSH:
     ```bash
     ssh -i "my-devec2key.pem" ec2-user@<NFS_SERVER_IP>
     ```
   - Use `lsblk` to list block devices and verify that the new EBS volumes are recognized.
   - Create partitions on each disk using `gdisk`:
     ```bash
     sudo gdisk /dev/xvdf
     sudo gdisk /dev/xvdg
     sudo gdisk /dev/xvdh
     ```
   - Install `lvm2` package and create physical volumes (PVs):
     ```bash
     sudo yum install lvm2 -y
     sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
     ```
   - Create a volume group (VG) named `webdata-vg` and add the PVs:
     ```bash
     sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
     ```
   - Create logical volumes (LVs) for apps, logs, and opt:
     ```bash
     sudo lvcreate -n lv-apps -L 9G webdata-vg
     sudo lvcreate -n lv-logs -L 9G webdata-vg
     sudo lvcreate -n lv-opt -L 9G webdata-vg
     ```
   - Format the LVs with the `xfs` filesystem:
     ```bash
     sudo mkfs -t xfs /dev/webdata-vg/lv-apps
     sudo mkfs -t xfs /dev/webdata-vg/lv-logs
     sudo mkfs -t xfs /dev/webdata-vg/lv-opt
     ```
   - Create mount points and mount the LVs:
     ```bash
     sudo mkdir /mnt/apps /mnt/logs /mnt/opt
     sudo mount /dev/webdata-vg/lv-apps /mnt/apps
     sudo mount /dev/webdata-vg/lv-logs /mnt/logs
     sudo mount /dev/webdata-vg/lv-opt /mnt/opt
     ```

3. **Install and configure the NFS server**.
   - Install NFS utilities and start the NFS server:
     ```bash
     sudo yum update -y
     sudo yum install nfs-utils -y
     sudo systemctl start nfs-server.service
     sudo systemctl enable nfs-server.service
     ```
   - Set permissions for the mounted directories:
     ```bash
     sudo chown -R nobody: /mnt/apps /mnt/logs /mnt/opt
     sudo chmod -R 777 /mnt/apps /mnt/logs /mnt/opt
     sudo systemctl restart nfs-server.service
     ```
   - Configure `/etc/exports` to allow access from the web server subnet:
     ```bash
     sudo vi /etc/exports

     /mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
     /mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
     /mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)

     sudo exportfs -arv
     ```
   - Open necessary ports in the security group: TCP 111, UDP 111, UDP 2049, NFS 2049.

### Step 2: Configure the Database Server (Ubuntu Linux)

1. **Launch an EC2 instance with Ubuntu Linux**.
   - Create an EC2 instance with Ubuntu Linux as the operating system.
   - Ensure it is in the same VPC and subnet as the NFS and web servers.

2. **Install MySQL Server**.
   - Connect to the instance via SSH:
     ```bash
     ssh -i "my-devec2key.pem" ubuntu@<DB_SERVER_IP>
     ```
   - Update the system and install MySQL server:
     ```bash
     sudo apt update && sudo apt upgrade -y
     sudo apt install mysql-server -y
     ```
   - Secure the MySQL installation:
     ```bash
     sudo mysql_secure_installation
     ```

3. **Configure the MySQL database**.
   - Access the MySQL shell and create the `tooling` database:
     ```sql
     sudo mysql

     CREATE DATABASE tooling;
     CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
     GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.0.0/20' WITH GRANT OPTION;
     FLUSH PRIVILEGES;
     exit;
     ```
   - Update the MySQL configuration to allow connections from the web server subnet by setting the bind address:
     ```bash
     sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
     # Set the bind address to 0.0.0.0 or the server's private IP

     sudo systemctl restart mysql
     ```

4. **Open MySQL port 3306**.
   - Configure the security group to allow inbound traffic on port 3306 from the web server subnet.

### Step 3: Prepare the Web Servers (RHEL 9)

1. **Launch three EC2 instances with RHEL 9**.
   - Create three EC2 instances, each with RHEL 9, to act as web servers.
   - Ensure these instances are in the same VPC and subnet as the NFS and database servers.

2. **Install NFS client on all web servers**.
   - Connect to each instance via SSH and install the NFS client:
     ```bash
     sudo yum install nfs-utils nfs4-acl-tools -y
     ```
   - Mount the NFS export for apps:
     ```bash
     sudo mkdir /var/www
     sudo mount -t nfs -o rw,nosuid <NFS_SERVER_PRIVATE_IP>:/mnt/apps /var/www
     ```
   - Verify the NFS mount and ensure persistence after reboot:
     ```bash
     sudo vi /etc/fstab

     <NFS_SERVER_PRIVATE_IP>:/mnt/apps /var/www nfs defaults 0 0
     ```

3. **Install Apache and PHP**.
   - Install Apache and PHP with required modules:
     ```bash
     sudo yum install httpd -y
     sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
     sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.2
     sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
     ```
   - Start and enable Apache and PHP services:
     ```bash
     sudo systemctl start php-fpm
     sudo systemctl enable php-fpm
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```

4. **Mount Apache logs to NFS**.
   - Mount the Apache logs directory to the NFS export for logs:
     ```bash
     sudo mkdir /var/log/httpd
     sudo mount -t nfs -o rw,nosuid <NFS_SERVER_PRIVATE_IP>:/mnt/logs /var/log/httpd
     sudo vi /etc/fstab

     <NFS_SERVER_PRIVATE_IP>:/mnt/logs /var/log/httpd nfs defaults 0 0
     ```

5. **Deploy the Tooling Application**.
   - Fork the repository from GitHub and clone it into the web server's `/var/www/html` directory:
     ```bash
     sudo yum install git -y
     cd /var/www/html
     git clone https://github.com/yourusername/tooling.git .
     ```
   - Ensure that the necessary permissions are set for the `html` folder and its contents.

6. **Configure the application to connect to the database**.
   - Update the `functions.php` file with the correct database connection details:
     ```bash
     sudo vi /var/www/html/functions.php
     ```
   - Execute the `tooling-db.sql` script to populate the database:
     ```bash
     sudo mysql -h <DB_SERVER

_PRIVATE_IP> -u webaccess -p tooling < /var/www/html/tooling-db.sql
     ```

### Step 4: Load Balancer Configuration (Optional)

1. **Create an Elastic Load Balancer (ELB)**.
   - Go to the AWS Management Console, navigate to EC2, and create an Application Load Balancer.
   - Add the three web servers to the ELB target group.
   - Associate the ELB with the web servers' security group and allow HTTP/HTTPS traffic.

2. **Test the Setup**.
   - Access the application via the load balancer’s DNS name:
     ```bash
     http://<ELB_DNS_NAME>
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
