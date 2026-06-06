## 概述

LAMP 是指 Linux + Apache + MySQL + PHP 的完整网站服务器环境。本教程将指导您在 Ubuntu 系统上搭建完整的 LAMP 环境。

## 系统要求

- Ubuntu 18.04 或更高版本
- 具有 sudo 权限的用户账户
- 稳定的网络连接

## 步骤 1：更新系统包

```shell
sudo apt update
sudo apt upgrade -y
```

## 步骤 2：安装 Apache

### 安装 Apache2

```shell
sudo apt install apache2 -y
```

### 启动并设置开机自启

```shell
sudo systemctl start apache2
sudo systemctl enable apache2
```

### 验证安装

在浏览器中访问 `http://your-server-ip`，如果看到 Apache 默认页面，说明安装成功。

### 调整防火墙（如启用）

```shell
sudo ufw allow 'Apache Full'
```

## 步骤 3：安装 MySQL

### 安装 MySQL Server

```shell
sudo apt install mysql-server -y
```

### 安全配置 MySQL

```shell
sudo mysql_secure_installation
```

按照提示完成以下安全设置：

- 设置 root 密码
- 移除匿名用户
- 禁止 root 远程登录
- 移除测试数据库
- 重新加载权限表

### 启动并设置开机自启

```shell
sudo systemctl start mysql
sudo systemctl enable mysql
```

## 步骤 4：安装 PHP

### 安装 PHP 及常用扩展

```shell
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
```

### 配置 PHP

编辑 PHP 配置文件：

```shell
sudo nano /etc/php/7.4/apache2/php.ini
```

（注意：版本号可能不同，请根据实际安装的 PHP 版本调整）

建议修改以下配置：

```ini
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
max_execution_time = 300
date.timezone = Asia/Shanghai
```

### 重启 Apache 使配置生效

```shell
sudo systemctl restart apache2
```

## 步骤 5：测试 PHP

### 创建测试文件

```shell
sudo nano /var/www/html/info.php
```

### 添加以下内容

```php
<?php
phpinfo();
?>
```

### 访问测试

在浏览器中访问 `http://your-server-ip/info.php`，应该能看到 PHP 信息页面。

**重要**：测试完成后请删除此文件以确保安全：

```shell
sudo rm /var/www/html/info.php
```

## 步骤 6：配置虚拟主机（可选）

### 创建网站目录

```shell
sudo mkdir -p /var/www/example.com/public_html
```

### 设置目录权限

```shell
sudo chown -R www-data:www-data /var/www/example.com
sudo chmod -R 755 /var/www/example.com
```

### 创建虚拟主机配置文件

```shell
sudo nano /etc/apache2/sites-available/example.com.conf
```

### 添加以下内容

```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    <Directory /var/www/example.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### 启用站和重写模块

```shell
sudo a2ensite example.com.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

## 步骤 7：安装 phpMyAdmin（可选）

### 安装 phpMyAdmin

```shell
sudo apt install phpmyadmin -y
```

在安装过程中：

- 选择 `apache2` 作为 web 服务器
- 选择 `是` 来配置 dbconfig-common
- 设置 phpMyAdmin 的数据库密码

### 创建符号链接（如未自动创建）

```shell
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

### 访问 phpMyAdmin

在浏览器中访问 `http://your-server-ip/phpmyadmin`

## 步骤 8：安全加固

### 配置 MySQL 远程访问（如需要）

```shell
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

将 `bind-address` 从 `127.0.0.1` 改为 `0.0.0.0`（如需要远程访问）

### 创建专用 MySQL 用户

```sql
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
FLUSH PRIVILEGES;
```

## 常用管理命令

### Apache 服务管理

```shell
sudo systemctl start apache2      # 启动
sudo systemctl stop apache2       # 停止
sudo systemctl restart apache2    # 重启
sudo systemctl reload apache2     # 重载配置
sudo systemctl status apache2     # 查看状态
```

### MySQL 服务管理

```shell
sudo systemctl start mysql        # 启动
sudo systemctl stop mysql         # 停止
sudo systemctl restart mysql      # 重启
sudo systemctl status mysql       # 查看状态
```

## 故障排除

### 检查服务状态

```shell
sudo systemctl status apache2
sudo systemctl status mysql
```

### 查看日志文件

```shell
# Apache 错误日志
sudo tail -f /var/log/apache2/error.log

# MySQL 错误日志
sudo tail -f /var/log/mysql/error.log
```

### 测试 PHP 配置

```shell
php -v                          # 查看 PHP 版本
php -m                          # 查看已加载的模块
sudo apache2ctl configtest      # 测试 Apache 配置
```

## 总结

至此，您已在 Ubuntu 系统上成功搭建了 LAMP 环境。您现在可以开始部署网站应用程序了。记得定期更新系统和软件包以确保安全性。

**注意**：在生产环境中，请务必配置适当的安全措施，包括防火墙、SSL 证书和定期备份。