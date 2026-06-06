#概述
LNMP 是指 Linux、Nginx、MySQL（或 MariaDB）、PHP 组成的服务器架构，广泛用于部署动态网站。以下是基于 Ubuntu 20.04/22.04 系统的详细搭建步骤。
#一、前期准备
##更新系统包
确保系统软件包为最新版本：
```shell
sudo apt update && sudo apt upgrade -y
```
##安装必要依赖
```shell
sudo apt install -y curl wget vim
```
##关闭防火墙（可选，新手推荐）
生产环境建议配置具体规则，测试环境可临时关闭：
```shell
sudo ufw disable
```
#二、安装 Nginx（网页服务器）
Nginx 负责处理 HTTP 请求并返回网页内容，是 LNMP 架构的前端服务器。
##1.安装 Nginx
```shell
sudo apt install -y nginx
```
##2.启动并设置开机自启
```shell
sudo systemctl start nginx
sudo systemctl enable nginx
```
##3.验证安装
- 查看 Nginx 状态：sudo systemctl status nginx（显示 active (running) 即为正常）
- 在浏览器访问服务器 IP，若显示 "Welcome to nginx!" 页面，说明安装成功
#三、安装 MySQL / MariaDB（数据库）
Ubuntu 默认源中已用 MariaDB 替代 MySQL，两者兼容性良好，这里以 MariaDB 为例。
##1.安装 MariaDB
```shell
sudo apt install -y mariadb-server
```
##2.启动并设置开机自启
```shell
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
##3.安全配置（重要）
运行官方安全脚本，设置 root 密码、禁用匿名用户等：
```shell
sudo mysql_secure_installation
```
按提示操作：
- 输入当前 root 密码（初始为空，直接回车）
- 选择设置 root 密码（输入 Y 并设置强密码）
- 依次输入 Y 禁用匿名用户、禁止 root 远程登录、删除测试数据库
##4.验证登录
```shell
mysql -u root -p
```
输入设置的密码，成功进入 MariaDB 终端即表示正常（输入 exit 退出）。
#四、安装 PHP（动态脚本处理器）
PHP 负责处理动态内容，需安装 PHP-FPM（FastCGI 进程管理器）与 Nginx 配合。
##1.安装 PHP 及必要扩展
Ubuntu 20.04 默认 PHP 版本为 7.4，22.04 为 8.1，这里安装常用扩展：
```shell
sudo apt install -y php-fpm php-mysql php-cli php-gd php-curl php-mbstring php-xml
```
##2.查看 PHP 版本
```shell
php -v
```
##3.启动并设置开机自启
```shell
sudo systemctl start php8.1-fpm  # 注意版本号，如 php7.4-fpm
sudo systemctl enable php8.1-fpm
```
##4.验证 PHP-FPM 状态
```shell
sudo systemctl status php8.1-fpm
```
#五、配置 Nginx 解析 PHP
Nginx 本身不处理 PHP 代码，需通过配置将 .php 请求转发给 PHP-FPM 处理。
##1.创建网站目录
```shell
sudo mkdir -p /var/www/lnmp-test
sudo chown -R $USER:$USER /var/www/lnmp-test  # 赋予当前用户权限
```
##2.创建 Nginx 配置文件
```shell
sudo vim /etc/nginx/sites-available/lnmp-test
```
粘贴以下内容（注意修改 PHP 版本号）：
```conf
nginx
server {
    listen 80;
    server_name 你的服务器IP或域名;  # 如 192.168.1.100 或 example.com
    root /var/www/lnmp-test;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # 处理 PHP 请求
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;  # 注意PHP版本号
    }

    # 禁止访问 .htaccess 文件
    location ~ /\.ht {
        deny all;
    }
}
```
##3.启用配置并测试
```shell
sudo ln -s /etc/nginx/sites-available/lnmp-test /etc/nginx/sites-enabled/
sudo nginx -t  # 检查配置是否有误
sudo systemctl restart nginx  # 重启 Nginx 生效
```
#六、测试 LNMP 环境
##1.创建 PHP 测试文件
```shell
echo "<?php phpinfo(); ?>" > /var/www/lnmp-test/info.php
```
##2.访问测试页面
在浏览器输入 `http://你的服务器IP/info.php`，若显示 PHP 信息页面（包含版本、模块等信息），说明 LNMP 环境搭建成功。
##3.删除测试文件（安全建议）
测试完成后删除 info.php：
```shell
rm /var/www/lnmp-test/info.php
```
七、常用命令汇总
| 服务                                                         | 启动                              | 停止                             | 重启                                | 状态查看                           |
| ------------------------------------------------------------ | --------------------------------- | -------------------------------- | ----------------------------------- | ---------------------------------- |
| Nginx                                                        | `sudo systemctl start nginx`      | `sudo systemctl stop nginx`      | `sudo systemctl restart nginx`      | `sudo systemctl status nginx`      |
| MariaDB                                                      | `sudo systemctl start mariadb`    | `sudo systemctl stop mariadb`    | `sudo systemctl restart mariadb`    | `sudo systemctl status mariadb`    |
| PHP-FPM                                                      | `sudo systemctl start php8.1-fpm` | `sudo systemctl stop php8.1-fpm` | `sudo systemctl restart php8.1-fpm` | `sudo systemctl status php8.1-fpm` |
| 至此，Ubuntu 系统的 LNMP 环境已搭建完成，可用于部署 WordPress、Discuz 等 PHP 应用。 |                                   |                                  |                                     |                                    |