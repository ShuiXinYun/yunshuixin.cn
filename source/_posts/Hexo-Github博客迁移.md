---
title: Hexo+Github博客迁移
date: 2018-04-04 19:50:41
tags:
categories: Notes
---
之前博客用的是WordPress，部署在阿里云服务器上。阿里云学生优惠是10元/月，没优惠要100+/月，还是小水管，马上优惠要过期了，趁着有空换成Hexo+Github，这样只用花云存储的钱，省了服务器的花销。Hexo使用很简便，并且支持Markdown，配合Github用来建博客很合适,写东西比WordPress方便多了。
老博客:
<img class="aligncenter " src="/images/Hexo+Github博客迁移/old_blog.png" width=720/>
新博客:
<img class="aligncenter " src="/images/Hexo+Github博客迁移/new_blog.png" width=720/>
Hexo安装配置如下：
## Install npm, nodejs, Hexo
```bash
sudo apt-get install git
sudo apt-get install npm
```
### 安装 Node.js
```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
nvm install stable
```
### 安装 Hexo
```bash
npm install -g hexo-cli
npm install hexo-server --save
npm install --save hexo-deployer-git
npm install --save hexo-pdf
```
## Init
```bash
hexo init <blog>
cd <blog>
npm install
```
## [写作](https://hexo.io/zh-cn/docs/writing.html)

Preview:
```bash
hexo server 
```
## Install Theme:
```bash
cd <blog>
git clone --branch v5.1.2 https://github.com/iissnan/hexo-theme-next themes/next
```
## [生成](https://hexo.io/zh-cn/docs/generating.html)
## [部署](https://hexo.io/zh-cn/docs/deployment.html)
## Others
DNS解析设置:192.30.252.153
```bash
touch <blog>/source CNAME
echo your_website >> CNAME
```

