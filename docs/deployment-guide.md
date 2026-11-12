# PrestaShop AWS Deployment Guide

## 1. Introduction

This guide documents the deployment of PrestaShop 9.2.1 on AWS using two separate EC2 instances. One instance hosts the web application, while the other hosts the MariaDB database.

## 2. AWS Infrastructure Setup

### Region
- AWS Region: Europe (Stockholm)
- Region Code: eu-north-1

### Application Server
- Instance name: PrestaShop-App-Server
- Operating System: Amazon Linux 2023
- Instance type: t3.micro
- Storage: 8 GiB gp3
- Private IP: 172.31.32.125

### Database Server
- Instance name: PrestaShop-Database-Server
- Operating System: Amazon Linux 2023
- Instance type: t3.micro
- Storage: 8 GiB gp3
- Private IP: 172.31.16.95

Both instances were deployed in the same VPC to enable private communication.

## 3. Security Group Configuration

Two separate Security Groups were configured.

### Application Security Group
- SSH – TCP 22, restricted to the administrator's public IP.
- HTTP – TCP 80, allowed from the internet.
- HTTPS – TCP 443 was not configured.

### Database Security Group
- SSH – TCP 22, restricted to the administrator's public IP.
- MySQL/Aurora – TCP 3306, restricted to the application server's Security Group.

The database port was not exposed to the public internet.

## 4. Application Server Installation

Connected to the application server using SSH.

Installed Apache:

```bash
sudo dnf update -y
sudo dnf install -y httpd
sudo systemctl enable --now httpd
```

Installed PHP and the required extensions:

```bash
sudo dnf install -y php8.5 php8.5-cli php8.5-modphp php8.5-mysqlnd php8.5-gd php8.5-intl php8.5-mbstring php8.5-pdo php8.5-xml php8.5-zip php8.5-bcmath php8.5-sodium
```

Enabled PHP-FPM:

```bash
sudo systemctl enable --now php-fpm
```

The PHP memory limit was increased to 512 MB.

## 5. Database Server Installation

Connected separately to the database server using SSH.

Installed MariaDB:

```bash
sudo dnf install -y mariadb1011-server
sudo systemctl enable --now mariadb
```

Ran the MariaDB secure installation procedure to:
- Remove anonymous database users.
- Remove the test database.
- Disable remote root login.
- Reload privilege tables.

Configured MariaDB to listen on the database server's private IP address.

## 6. Database Creation

Created the PrestaShop database:

```sql
CREATE DATABASE prestashop
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

Created a dedicated database user named `prestashop_user`, restricted to connections from the application server's private IP address.

Granted the user privileges on the PrestaShop database.

The actual database password is intentionally excluded from this guide.

## 7. PrestaShop Deployment

Downloaded PrestaShop 9.2.1 from the official PrestaShop website.

Transferred the installation archive to the application server and extracted the files into:

```bash
/var/www/html
```

Configured ownership and permissions:

```bash
sudo chown -R apache:apache /var/www/html

sudo find /var/www/html -type d -exec chmod 755 {} \;

sudo find /var/www/html -type f -exec chmod 644 {} \;

sudo restorecon -RFv /var/www/html
```

Opened the application server's public IP address in a browser to launch the PrestaShop installation wizard.

## 8. Database Connection

During the PrestaShop installation, the following database details were supplied:

| Setting | Value |
|---|---|
| Database Server | 172.31.16.95 |
| Database Name | prestashop |
| Database User | prestashop_user |
| Database Port | 3306 |
| Table Prefix | ps_ |

The database password was entered privately during installation and is not included in this repository.

## 9. Memory Optimisation

During installation, the application server experienced memory pressure.

A 2 GiB swap file was configured:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

The swap configuration was added to `/etc/fstab` to persist after reboot.

PHP-FPM was also adjusted to use an on-demand worker configuration:

```ini
pm = ondemand
pm.max_children = 2
pm.process_idle_timeout = 10s
pm.max_requests = 200
```

The PHP-FPM configuration was tested and the services restarted.

## 10. Testing and Verification

The following services were checked:

```bash
sudo systemctl status httpd php-fpm --no-pager
```

On the database server:

```bash
sudo systemctl status mariadb --no-pager
```

All three services were confirmed active and running.

The PrestaShop storefront was accessed through:

http://13.60.250.26

The administration login page was also verified.

## 11. Final Outcome

PrestaShop was successfully deployed on AWS using a two-tier architecture.

The application and database were hosted on separate EC2 instances, private database communication was configured, and the storefront and administration login page were verified.

## 12. Security Notes

- Database access was restricted to the application server's Security Group.
- SSH access was restricted to the administrator's public IP.
- Database root access was not enabled remotely.
- No passwords, private keys or access tokens are included.
- HTTPS was not configured; the deployment uses HTTP and is intended for demonstration purposes.