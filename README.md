#PHP-

1.项目结构搭建
mkdir lamp-project && cd lamp-project
mkdir -p conf/apache conf/php htdocs build

2.准备 PHP 脚本（测试用）
Vim htdocs/index.php

<?php
phpinfo();
echo "<h1>Hello from PHP Container!</h1>";
?>

3.编写 Dockerfile（构建 PHP 镜像）
Vim build/Dockerfile



FROM php:7.4-apache
# 安装PHP扩展（如MySQL驱动、PDO）
RUN apt-get update && apt-get install -y \
    libmariadb-dev-compat \
    libmariadb-dev \
    && docker-php-ext-install mysqli pdo pdo_mysql
# 启用Apache重写模块
RUN a2enmod rewrite
# 启用MPM模块（这里以mpm_prefork为例）
RUN a2enmod mpm_prefork
# 设置Apache文档根目录
ENV APACHE_DOCUMENT_ROOT /var/www/html
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/ports.conf



4.配置 Docker Compose 文件
Vim docker-compose.yml

version: '3.1'
services:
  web:
    build: ./build          # 使用自定义Dockerfile构建镜像
    container_name: lamp-web
    restart: always
    ports:
      - "8080:80"          # 映射主机8080端口到容器80端口
    volumes:
      - ./htdocs:/var/www/html       # 挂载本地网页目录到容器
#     - ./conf/apache/apache2.conf:/etc/apache2/apache2.conf:ro  # 挂载Apache配置（只读）
  db:
    image: mysql:5.7        # 使用MySQL 5.7镜像
    container_name: lamp-mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=Abc_123   # 数据库root密码
      - MYSQL_DATABASE=php_db         # 创建数据库
      - MYSQL_USER=tester             # 普通用户
      - MYSQL_PASSWORD=abc123         # 普通用户密码
    volumes:
      - db_data:/var/lib/mysql       # 持久化数据库数据
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest  # 使用phpMyAdmin管理工具
    container_name: lamp-admin
    restart: always
    ports:
      - "8088:80"         # 映射phpMyAdmin端口
    environment:
      - PMA_HOST=db       # 指定数据库主机（与Docker Compose服务名一致）
      - MYSQL_USER=tester
      - MYSQL_PASSWORD=abc123
volumes:
  db_data:  # 定义数据卷

5.安装 Docker Compose(如果已安装直接第六步)

下载 Docker Compose 二进制文件
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

赋予执行权限
sudo chmod +x /usr/local/bin/docker-compose
验证安装
docker-compose --version
