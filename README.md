


# Hosting a WordPress Website on AWS

This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

## Architecture Overview

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- **VPC (Virtual Private Cloud):** Contains public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- **Internet Gateway:** Allows communication between instances in the VPC and the internet.
- **Security Groups:** Acts as a virtual firewall to control inbound and outbound traffic.
- **Public Subnets:** Used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- **Private Subnets:** Hosts web servers to enhance security.
- **EC2 Instance Connect Endpoint:** Provides secure SSH access to instances.
- **Application Load Balancer (ALB):** Distributes incoming web traffic across multiple EC2 instances.
- **Auto Scaling Group (ASG):** Automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
- **Amazon RDS (Relational Database Service):** Provides a managed relational database service for WordPress.
- **Amazon EFS (Elastic File System):** A scalable, elastic file storage system for WordPress.
- **AWS Certificate Manager (ACM):** Manages SSL/TLS certificates for secure communication.
- **Amazon SNS (Simple Notification Service):** Sends notifications related to Auto Scaling Group activities.
- **Amazon Route 53:** Manages domain names and DNS for routing traffic to the WordPress site.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, mounting Amazon EFS, and configuring WordPress with your RDS details.

```bash
# create to root user
sudo su

# update the software packages on the EC2 instance 
sudo yum update -y

# create an html directory 
sudo mkdir -p /var/www/html

# environment variable. Paste your EFS DNS name here
EFS_DNS_NAME=fs-02d3263459aa2a318.efs.us-east-1.amazonaws.com

# mount the EFS to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install PHP 8 along with necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# edit the wp-config.php file with your RDS details
sudo vi /var/www/html/wp-config.php

# restart the webserver
sudo service httpd restart
```

### Want a faster and error-free workflow?
Add your EFS DNS name to the script, copy the script up until the section where you create the `wp-config.php` file, and paste that in your EC2 launch template. This way, the only thing you will have to manually do after your EC2 is available is edit the `wp-config.php` file and restart the web server. Remember to check the file to ensure everything is perfect.

### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group. It ensures that new instances are correctly configured with the necessary software and settings.

```bash
#!/bin/bash
# update the software packages on the EC2 instance 
sudo yum update -y

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install PHP 8 along with necessary extensions for WordPress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install MySQL version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# environment variable. Paste your EFS DNS name here
EFS_DNS_NAME=fs-02d3263459aa2a318.efs.us-east-1.amazonaws.com

# mount the EFS to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# set permissions
chown apache:apache -R /var/www/html

# restart the webserver
sudo service httpd restart
```

## How to Use

1. Clone this repository to your local machine.
2. Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.) as outlined in the architecture overview.
3. Use the provided scripts to set up the WordPress application on EC2 instances within the VPC.
4. Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
5. Access the WordPress website through the Load Balancer's DNS name.

---

This setup ensures that your WordPress website is hosted in a highly available and scalable environment, leveraging AWS services like EC2, RDS, EFS, and more. Make sure to adjust configurations as needed for your specific requirements.
