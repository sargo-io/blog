# Sargo Blog

A customizable and user-friendly blog website built with wordpress. This website allows users to read, create, edit, and delete posts. The blog is designed for easy content management and engaging user interaction.

## Features

- **Responsive Design**: Optimized for desktops, tablets, and mobile devices.
- **Admin Dashboard**: Manage posts, users, and settings with an admin panel.

## Getting Started

This document outlines the steps taken to launch the Sargo WordPress site on an AWS ECS instance using Amazon Linux and Apache and the updating the themes used in the site.

## Sections

1. **Launching WordPress on AWS ECS with Amazon Linux and Apache**
2. **Updating the theme**

# 1. Launching WordPress on AWS ECS with Amazon Linux and Apache

This sectin outlines the steps to launch a WordPress site on an AWS ECS instance using Amazon Linux and Apache.

## Prerequisites

1. **AWS Account**: Ensure you have an AWS account with necessary permissions.
2. **IAM Roles**: Create an IAM role for ECS tasks with permissions for Amazon EC2, if required.
3. **AWS CLI**: Install and configure the AWS CLI with appropriate access.
4. **Security Groups**: Configure security groups with inbound rules for HTTP (80), HTTPS (443), and SSH (22).

---

## Step 1: Set Up ECS Cluster and Task Definition

1. **Launch ECS Cluster**:

   - Go to the **ECS Dashboard** and create a new ECS Cluster.
   - Select the **EC2 Linux + Networking** option and configure the instance size, number, and network settings.
   - Assign a security group with HTTP and SSH access.
   - Download the "accessKeyPair.pem" using your credeatials fro SSH access.

2. **Create a Task Definition**:
   - Go to **Task Definitions** in ECS and create a new EC2 task definition.
   - Assign the ECS role created earlier and configure the task’s memory and CPU requirements.
   - Add any required environment variables or storage configurations.

---

## Step 2: Configure EC2 Instance with Amazon Linux

1. **Connect to EC2 Instance**:

- From the EC2 Dashboard, connect to the instance via SSH.

  ```bash
  ssh -i /path/to/your-key.pem ec2-user@your-ecs-instance-public-ip
  ```

  Replace `/path/to/your-key.pem` with the path to your EC2 key pair and `your-ecs-instance-public-ip` with your instance's public IP address.

2. **Update Packages**:

```bash
   sudo yum update -y
```

3. **Install and Start Apache Web Server**:

```bash
   sudo yum install -y httpd
   sudo service httpd start
   sudo chkconfig httpd on
```

4. **Install PHP and MySQL Client**:

- Install PHP for WordPress compatibility.

```bash
sudo dnf upgrade -y
sudo dnf install -y httpd wget php-fpm php-mysqli php-json php php-devel
sudo dnf install mariadb105-server

```

---

## Step 3: Configure Database

1. **Create an RDS Database**:

   - Set up an RDS instance with MySQL.
   - Note the database endpoint, username, and password for configuring WordPress.

2. **Configure Security Group**:
   - Add the ECS instance’s security group to the RDS instance's inbound rules for MySQL access.

---

## Step 4: Download and Configure WordPress

1. **Download WordPress**:

   ```bash
   cd /var/www/html
   sudo wget https://wordpress.org/latest.tar.gz
   sudo tar -xzf latest.tar.gz
   sudo mv wordpress/* .
   sudo rm -rf wordpress latest.tar.gz
   ```

2. **Configure Permissions**:

   - Change ownership to Apache user.
     ```bash
     sudo chown -R apache:apache /var/www/html
     ```
   - Set permissions to make WordPress files editable by Apache.
     ```bash
     sudo chmod -R 755 /var/www/html
     ```

3. **Configure WordPress Database**:
   - Rename `wp-config-sample.php` to `wp-config.php` and update database details.
     ```bash
     sudo cp wp-config-sample.php wp-config.php
     sudo nano wp-config.php
     ```
   - Edit the following values:
     ```php
     define('DB_NAME', 'database_name');
     define('DB_USER', 'database_user');
     define('DB_PASSWORD', 'database_password');
     define('DB_HOST', 'database_host');
     ```
   - Save and exit.

---

## Step 5: Security and Performance Tuning

1. **Configure Security Settings**:

   - Set permissions for sensitive files:
     ```bash
     sudo chmod 600 /var/www/html/wp-config.php
     ```

This setup will launch WordPress on an ECS instance with Amazon Linux and Apache, ready for customization and content addition.

# 2. Updating themes

This guide walks through the steps to update a WordPress theme by connecting via SSH.

## Steps

### 1. Transfer the theme file to /tmp folder in aws ecs

1. **Open your terminal** and connect to your ECS instance and transfer the theme.zip file using SCP:

   ```bash
    scp -i `/path/to/your-key.pem` `/path/to/theme.zip``your-ecs-instance-public-ip`:/tmp/
   ```

   Replace `/path/to/your-key.pem` with the path to your EC2 key pair and `your-ecs-instance-public-ip` with your instance's public IP address.

---

### 2. Connect to Your ECS Instance

1. **Open your terminal** and connect to your ECS instance using SSH:

   ```bash
   ssh -i /path/to/your-key.pem ec2-user@your-ecs-instance-public-ip
   ```

   Replace `/path/to/your-key.pem` with the path to your EC2 key pair and `your-ecs-instance-public-ip` with your instance's public IP address.

2. **Navigate to the WordPress directory**:
   ```bash
   cd /var/www/html/wp-content/themes
   ```
   This is where WordPress themes are stored.

---

### 3. Install the Theme

Once in the wp-content/themes folder,
Move the zip file from /tmp

```bash
    mv /tmp/theme.zip .
```

Unzip the theme

```bash
    unzip theme.zip
```

### 4. Set Permissions for the Updated Theme

1. Ensure the new theme files have the correct ownership and permissions:

   ```bash
   sudo chown -R apache:apache /var/www/html/wp-content/themes/theme
   sudo chmod -R 755 /var/www/html/wp-content/themes/theme
   ```

2. **Restart the Apache server** to apply changes:
   ```bash
   sudo systemctl restart httpd
   ```

---
