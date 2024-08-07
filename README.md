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

## Step 1 - Prepare NFS Server

