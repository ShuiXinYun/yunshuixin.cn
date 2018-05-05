---
title: WordPress个人博客搭建
date: 2017-07-20
tags:
categories: Notes
---
# Step 1 域名购买、服务器deploy
域名可直接从阿里云购买，同时可购买域名解析服务. 服务器可选择搬瓦工、Vultr、阿里云等。阿里云针对学生有优惠，但对于普通用户价格较贵（100+/月），带宽低配1mbps，延迟低。Vultr的延迟情况一般，有亚洲节点（日本、新加坡）一个月5美元的配置应当能够满足个人博客要求；搬瓦工最便宜，低配一个月3美元，但没有亚洲节点，延迟较低的服务器是洛杉矶，延迟大概在200ms。搬瓦工、Vultr的带宽都要比阿里云大得多。在阿里云的控制台中将域名和服务器IP地址绑定: 万网域名解析设置方法

#Step 2 服务器端配置
## 更新
使用putty/xshell远程ssh连接
```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python-pip
```

##[安装LNMP](https://lnmp.org/install.html)
```shell
apt-get install screen
screen -S lnmp
wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz && tar zxf lnmp1.4.tar.gz && cd lnmp1.4 && ./install.sh lnmp

```
+ 目前提供了较多的MySQL、MariaDB版本和不安装数据库的选项，需要注意的是MySQL 5.6,5.7及MariaDB 10必须在1G以上内存的更高配置上才能选择，不确定可直接选default.
+ 之后设置password（设置过程中删除需按住ctrl加backspace）
InnoDB引擎默认为开启，一般建议开启
+ PHP版本选择
+ 是否安装内存优化，默认不安装
+ install开始，视情况需要几十分钟. 如果显示Nginx: OK，MySQL: OK，PHP: OK，并且Nginx、MySQL、PHP都是running，80和3306端口都存在，并提示安装使用的时间及Install lnmp V1.4 completed! enjoy it.的话，说明已经安装成功。

## [安装虚拟主机](https://lnmp.org/faq/lnmp-vhost-add-howto.html)
```shell
lnmp vhost add
输入域名，添加更多域名（带www和不带表示不同的域名，可将这两个加入）
设置网站目录，回车采用默认：/home/wwwroot/域名
Allow rewrite rule:Y
enter the rewrite of programme:wordpress
Allow access log:Y
enter ascess log filename:回车，默认日志目录为：/home/wwwlogs/ 默认文件名为：域名.log
Creat database and MySQL user with same name? 选Y或N均可
add ssl certificate? 选择N，
虚拟主机配置文件在：/usr/local/nginx/conf/vhost/域名.conf，nano打开，在root/home/wwwroot/www.vpser.net;这一行下面添加：include wordpress.conf;
```
主机删除：
```shell
lnmp vhost del
chattr -i /网站目录/.user.ini
sudo rm -rf 目录名
```

## 安装WordPress
```shell
cd ~
wget http://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
```
+ 在一键安装LNMP的时候已经安装了phpMyAdmin，通过[服务器ip]/phpmyadmin可以访问数据库后台。为了安全起见要改一下phpmyadmin这个文件夹的名字，它位于路径/home/wwwroot/default/
+ 使用phpMyAdmin建立wordpress数据库和权限，可百度phpMyAdmin使用方法。（阿里云要自定入站规则）
+ 配置wordpress
    ```shell
    cd ~ wordpress
    mv wp-config-sample.php wp-config.php
    nano wp-config.php
    ```
    + DB_NAME：在第二步中为WordPress创建的数据库名称
    + DB_USER：在第二步中创建的WordPress用户名
    + DB_PASSWORD：第二步中为WordPress用户名设定的密码
    + DB_HOST：第二步中设定的hostname（通常是localhost）
    + 将wordpress文件夹里的所有文件（不包括wordpress这个文件夹本身）移动到网站的根目录下，例如/home/wwwroot/你的域名文件夹。mv * /home/wwwroot/域名
    + wordpress切换中文：通过FTP、SSH等方式打开并编辑站点根目录下的wp-config.php文件，查找define(‘WPLANG’, ”);一行，在第二个参数处填入zh_CN，变成define(‘WPLANG’, ‘zh_CN’);并保存文件。进入站点控制板（dashboard），看到更新提示后进行升级即可。WordPress会自动从官方网站下载中文语言包并安装。
+ 访问域名，ENJOY ~.