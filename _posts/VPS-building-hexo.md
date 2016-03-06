title: VPS(CentOS)搭建Hexo博客与Git Hooks更新（小白篇）
date: 2016-3-2 22:28:00
categories:
- 环境搭建
tags:
- CentOS
- VPS
- Hexo
- Hooks
permalink: VPS-building-Hexo
---
　　本机使用Hexo生成静态文件，通过Git推送到VPS的Git仓库。VPS配置Git Hooks,将静态文件同步到站点目录，再配置Nginx静态环境解析站点目录文件，显示Blog页面。

<!-- more -->

## 写文章目的
1. 疏通VPS搭建Hexo的流程，理解每个搭建环节的作用。
2. 将过程记录下来，引导想搭建相同环境的朋友少走弯路。

## 搭建流程与环境介绍

- Hexo博客从生产到展示的过程
![流程图](/resources/VPS-And-Hexo/1.png)
- VPS环境简介：我用的是`搬瓦工` 的VPS，其Root-Shell-basic有限制
- VPS配置信息：
![VPS配置信息](/resources/VPS-And-Hexo/2.png)
- 本地配置信息：Mac 10.10.5
![本地配置信息](/resources/VPS-And-Hexo/3.png)

## 要点分类：
1. 本机安装Node.js、Hexo用于生成静态文件
2. 本机安装Git,将静态文件Push到VPS的Git仓库
3. VPS上安装Git，建立Git仓库、配置Hooks来同步静态文件
4. 购买域名，在VPS上安装及配置Nginx环境，解析站点

## 安装步骤

#### 本机安装软件

本机需要安装Node.js、Hexo、Git,还不会装软件的自行Google

#### 本地配置Git
> 本次演示配置信息如下，各自配置时替换即可
> 本地Git邮箱：hansoncoder@gmail.com
> 本地Git用户名：Hanson
> VPS上网站配置文件:hexo.conf
> VPS上站点目录：/var/www/hansoncoder.com/public
> VPS上网站管理用户：Hanson
> 域名：hansoncoder.com

1 设置本地Git邮箱及用户名

> ```
MacdeiMac-2:~ Mac$ git config --global user.email "hansoncoder@gmail.com” #邮箱
MacdeiMac-2:~ Mac$ git config --global user.name "Hanson" #用户名
```
> 查看配置是否正确
> ```
MacdeiMac-2:~ Mac$ git config --list
user.email=hansoncoder@gmail.com
user.name=Hanson
filter.lfs.clean=git-lfs clean %f
filter.lfs.smudge=git-lfs smudge %f
filter.lfs.required=true
```

2 生成sshkey
> ```
MacdeiMac-2:~ Mac$ ssh-keygen -t rsa -C "hansoncoder@gmail.com" #一路回车
MacdeiMac-2:~ Mac$ cat .ssh/id_rsa.pub #复制公钥内容，从AAA开始到结束
ssh-rsa AAAA....hansoncoder@gmail.com #...代替秘钥中间内容
```

#### 购买域名

![购买的域名](/resources/VPS-And-Hexo/4.png)

#### VPS上安装Nginx

补充命令知识：
> cat 查看文件内容
> ls -l 查看文件详细信息，包括权限
> echo 输出信息 （-e 是识别转义字符，输出的内容中"\n"被转义为换行）
> mkdir 创建目录(文件夹)
> touch 创建文件

1 配置Nginx源文件
> 在/etc/yum.repos.d目录下创建一个源配置文件nginx.repo,写入如下代码

> ```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```
> 执行命令: 以后写入文件默认用echo -e 指令，以后会忽略指令过程，在适当的地方需要反斜杠(\)作为转义字符，大家也可以用图形界面操作。
> ```
[root /]# cd /etc/yum.repos.d
[root yum.repos.d]# echo -e "
[nginx]\n
name=nginx repo\n
baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/\n
gpgcheck=0\n
enabled=1\n
" > nginx.repo
```
> 查看是否配置正确(使用cat 命令查看内容，以下配置文件大家自行使用查看命令验证配置信息)
> ```
[root /]# cat /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

2 安装Nginx

> ```
[root /]# yum install nginx -y
```
> 查看Nginx版本号(安装完成验证)
> ```
[root /]# nginx -v
nginx version: nginx/1.8.1
```

#### VPS上部署Nginx

1 添加站点解析配置文件

> 在/etc/nginx/sites-available目录下创建一个配置文件XXX.conf(这里以hexo.conf为例),并在文件中输入一下内容
> ```
server {
  root /var/www/hansoncoder.com/public; #站点目录
  index index.html index.htm; #主页，不用改
  server_name www.hansoncoder.com hansoncoder.com *.hansoncoder.com; #4.03 申请的域名
  location / {
    try_files $uri $uri/ /index.html;
  }
}
```
> 执行以下命令,然后通过图形化界面写入内容
> ```
[root /]# mkdir -p /etc/nginx/sites-available
[root /]# touch /etc/nginx/sites-available/hexo.conf
```
> ![图形用户写入内容过程](/resources/VPS-And-Hexo/5.png)
> 查看hexo.conf配置文件信息(配置验证)
> ![站点及域名(hexo.conf文件)配置信息](/resources/VPS-And-Hexo/6.png)

2 配置Nginx配置文件

> 在/etc/nginx/nginx.conf文件中添加如下代码(添加包含`sites-available`目录及站点`/var/www`目录下的配置文件)
> ```
include /etc/nginx/sites-available/*.conf;
include /var/www/*.conf;
```
> ![Nginx配置文件(nginx.conf)配置步骤](/resources/VPS-And-Hexo/9.png)
> 检查配置是否成功
> ![Nginx配置文件(nginx.conf)配置内容](/resources/VPS-And-Hexo/10.png)

3 配置防火墙

> ```
[root /]# iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```
> 防火墙配置验证：
> ```
[root /]# /etc/init.d/iptables status
```
> 显示结果如果有：`1 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:80\`，说明配置成功。

4 启动Nginx服务
> ```
[root /]# service nginx reload
[root /]# /etc/init.d/nginx start
```
> ![Nginx服务启动成功](/resources/VPS-And-Hexo/11.png)

#### 创建站点管理者

为了安全考虑，在VPS上创建一个普通用户(Hanson)专门来管理网站的,其用户组为sitesManagers。

1 创建用户组及用户
> 并创建其家目录与分配用户到sitesManagers组
> ```
[root /]# groupadd sitesManagers #创建用户组
[root /]# useradd Hanson -m -g sitesManagers #创建用户，
```
> 查看用户信息(验证创建用户是否成功)
> ```
[root /]# finger Hanson
Login: Hanson         			Name: 
Directory: /home/Hanson             	Shell: /bin/bash
Last login Wed Mar  2 05:39 (EST) on pts/0 from 113.108.199.76
No mail.
No Plan.
[root /]# groups Hanson
Hanson : sitesManagers
```

2 设置用户密码

> ```
passwd Hanson
```
> 设置密码成功如下
> ![Hanson密码设置成功](/resources/VPS-And-Hexo/13.png)

3 设置家目录权限

> ```
chmod 755 /home/Hanson
```

4 给Hanson用户添加sudo权限

> 修改/etc/sudoers文件，找到如下指令添加一条
> ```
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
Hanson   ALL=(ALL)     ALL #新增这条指令
```
> ![用户权限设置信息](/resources/VPS-And-Hexo/14.png)

#### 建站测试

1 创建测试页面
> 进入`/var/www/hansoncoder.com/public`目录（目录由4.0.5的root值确定）,这里涉及的切换用户，应该使用”Root shell -interactive“
> ```
[root@localhost /]# cd /var
[root@localhost var]# mkdir www
[root@localhost var]# chmod 777 www
[root@localhost var]# su Hanson 
[Hanson@localhost var]# mkdir -p /var/www/hansoncoder.com/public
[Hanson@localhost var]# cd /var/www/hansoncoder.com/public
[Hanson@localhost public]# echo “This is test HTML” > index.html
```

2 测试(打开浏览器，输入你的域名，看看能否看到：This is test HTML)
3 删除测试文件index.html

> ```
[Hanson@localhost public]# rm index.html
```
> ![站点目录信息](/resources/VPS-And-Hexo/15.png)

#### VPS上安装配置Git

1 安装并验证Git
> ```
[root@localhost /]# yum install
[root@localhost /]# git --version
git version 1.7.1
```

2 使用普通用户配置Git

> 在网站管理者的家目录配置.ssh公钥，用于静态文件本地与VPS同步。复制本地公钥，粘贴到VPS的authorized_keys文件中
> ```
[root@localhost /]# su Hanson
[Hanson@localhost /]# cd /Home/Hanson
[Hanson@localhost ~]# mkdir .ssh
[Hanson@localhost .ssh]# cd .ssh
[Hanson@localhost .ssh]# echo "本地的公钥(AAA开头)" > authorized_keys
[Hanson@localhost .ssh]# cat authorized_keys #查看是否添加成功
AAA....hansoncoder@gmail.com
```
> 无法使用shell-basic切换用户的，通过如下操作
> ![Hanson创建authorized_keys文件](/resources/VPS-And-Hexo/16.png)
> ![界面添加内容](/resources/VPS-And-Hexo/17.png)

3 设置VPS上ssh的端口号

> 将/etc/ssh/sshd_config文件中的Port为22设置如下：
> ```
Port 22
```

4 ssh远程登录验证

> 在`本地`终端登录VPS的Hanson用户(ssh 用户名@域名),成功登录如下: 
> ```
MacdeiMac-2: Mac$ ssh Hanson@hansoncoder.com
git@hansoncoder.com's password: 
Last login: Tue Mar  1 02:21:36 2016 from 113.108.199.76
[Hanson@localhost ][6]$ 
```

#### VPS上配置Git仓库

在VPS上初始化Git仓库，配置Hooks目录下的post-receive文件，实现本地静态文件发布时自动同步到站点目录

1 初始化Git仓库
> ```
[root@localhost /]# su Hanson
[Hanson@localhost /]# cd /home/Hanson
[Hanson@localhost ~]# mkdir HansonBlog.git
[Hanson@localhost ~]# cd HansonBlog.git
[Hanson@localhost HansonBlog.git]# git init —bare
[Hanson@localhost HansonBlog.git]# ls -l
```
> ![初始化成功信息](/resources/VPS-And-Hexo/18.png)

2 配置VPS上Git仓库的Hooks

> 在/home/Hanson/HansonBlog.git/Hooks目录创建post-receive,并写入以下内容
> ```
#!/bin/bash -l
GIT_REPO=/home/Hanson/HansonBlog.git #Git仓库 10.1确定
TMP_GIT_CLONE=/tmp/HansonBlog
PUBLIC_WWW=/var/www/hansoncoder.com/public #站点目录 5.2步确定
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```
> 查看 post-receive文件内容(验证Hooks配置信息)
> ![post-receive文件内容](/resources/VPS-And-Hexo/19.png)

## 博客初始化与编译发布

1. 本地初始化Hexo
```
MacdeiMac-2: Mac$ cd /Users/Mac/Desktop
MacdeiMac-2:Desktop Mac$ mkdir myBlog
MacdeiMac-2:Desktop Mac$ cd myBlog
MacdeiMac-2:myBlog Mac$ hexo init
```

2. Hexo编译生成静态文件
```
MacdeiMac-2:myBlog Mac$ npm install
MacdeiMac-2:myBlog Mac$ hexo g
```

3. 修改配置文件_config.yml,在_config.yml文件末写入如下内容
```
deploy:
type: git
message: Hanson
repo: Hanson@hansoncoder.com:HansonBlog.git
branch: master
```

4. 发布到VPS服务器
```
MacdeiMac-2:myBlog Mac$ npm install hexo-deployer-git --save
MacdeiMac-2:myBlog Mac$ hexo d
```

## 配置结束，发布文章

- 写markdown文件，然后保存到suource/_posts文件夹
- hexo g
- hexo d

## 问题与解决方案

1. 用`yum install git`安装Git出现问题：/bin/bash: git: command not found
解决（修改安装命令为）：yum install git git-svn git-email git-gui gitk -y

2. ssh配置后，ssh连接验证：ssh: connect to host localhost port 22: Connection refused
错误原因：
2.1. sshd 未安装：yum install openssh-server
2.2. sshd 未启动：chkconfig sshd on
2.3. 防火墙：chkconfig iptables off(永久性生效，重启后不会复原)
2.4. 端口号没有设置为22(或者设置后没有重启：service sshd restart)

3. 使用hero d报错：ERROR Deployer not found: git
解决：npm install hexo-deployer-git --save

## 参考连接

1. ***安装流程*** by **叶雨梧桐BLOG** on <code>2015/03/04</code>: <http://blog.gt520.com/vps/239.html>
1. ***linux用户管理*** by **海底苍鹰(tank)博客** on <code>2010/11/26</code>: <http://blog.51yip.com/linux/1137.html>
1. ***VPS搭建Hexo，用GIt更新*** by **WikiLibrary** on <code>2016/01/26</code>: <http://tiktoking.github.io/2016/01/26/hexo/>
1. ***CentOS上配置Ngxin实践*** by **王皓** on <code>2015/01/29</code>: <http://ninghao.net/blog/2088>