# Nginx学习笔记

- Nginx 是开源的轻量级 Web 服务器、反向代理服务器，以及负载均衡器和 HTTP 缓存器。其特点是高并发，高性能和低内存。
- Nginx 专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载的考验，最大能支持 50000 个并发连接数。
- 
  Nginx 还支持热部署，它的使用特别容易，几乎可以做到 7x24 小时不间断运行。Nginx 的网站用户有：百度、淘宝、京东、腾讯、新浪、网易等。

## 1 Nginx的功能

### 1.1 正向代理

​		Nginx 不仅可以做反向代理，实现负载均衡，还能用做正向代理来进行上网等功能。

<img src=".\imgs\640(1).png" style="zoom:100%;" />

### 1.2 反向代理

​		客户端对代理服务器是无感知的，客户端不需要做任何配置，用户只请求反向代理服务器，反向代理服务器选择目标服务器，获取数据后再返回给客户端。

​		反向代理服务器和目标服务器对外而言就是一个服务器，只是暴露的是代理服务器地址，而隐藏了真实服务器的 IP 地址。

<img src=".\imgs\640(2).png" style="zoom:100%;" />

### 1.3 负载均衡

​		将原先请求集中到单个服务器上的情况改为增加服务器的数量，然后将请求分发到各个服务器上，将负载分发到不同的服务器，即负载均衡。

<img src=".\imgs\640(3).png" style="zoom:100%;" />

### 1.4 动静分离

​		为了加快网站的解析速度，可以把静态页面和动态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力。

<img src=".\imgs\640(4).png" style="zoom:100%;" />

### 1.5 高可用

​		为了提高系统的可用性和容错能力，可以增加 Nginx 服务器的数量，当主服务器发生故障或宕机，备份服务器可以立即充当主服务器进行不间断工作。

<img src=".\imgs\640(5).png" style="zoom:100%;" />

## 2 安装Nginx

参考文章：http://nginx.org/en/linux_packages.html#RHEL-CentOS

①安装yum-utils

```
yum install yum-utils
```

②配置yum仓库，创建文件`/etc/yum.repos.d/nginx.repo`并且具有如下内容：

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

③安装nginx

```
yum install nginx
```

④测试

- 启动nginx

```bash
[root@localhost ~]# nginx
```

- 查看nginx进程状态

```bash
[root@localhost ~]# ps -ef | grep nginx
root       7508      1  0 23:01 ?        00:00:00 nginx: master process nginx
nginx      7509   7508  0 23:01 ?        00:00:00 nginx: worker process
root       7597   7388  0 23:09 pts/0    00:00:00 grep --color=auto nginx
```

- 查看防火墙状态

```bash
[root@localhost ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 三 2020-04-01 23:11:46 CST; 4s ago
     Docs: man:firewalld(1)
 Main PID: 7892 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─7892 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

4月 01 23:11:46 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewal....
4月 01 23:11:46 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall....
Hint: Some lines were ellipsized, use -l to show in full.
```

- 关闭防火墙

```bash
[root@localhost ~]# systemctl stop firewalld
```

- 查看服务器ip地址

```bash
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:96:ae:a3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.158/24 brd 192.168.5.255 scope global noprefixroute dynamic ens33
       valid_lft 1014sec preferred_lft 1014sec
    inet6 fe80::52f:55f9:4d3c:9847/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

- 在本地浏览器地址栏输入`http://192.168.5.158/`，可以看到以下页面说明您的nginx安装成功！

<img src=".\imgs\nginx测试.png" style="zoom:75%;" />

## 3 Nginx配置

### 3.1 配置文件说明

Nginx的配置文件位置一般为`···/nginx/conf/nginx.conf`

①文件结构

```json
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

Nginx 配置文件由三部分组成：

- **全局块，**主要设置一些影响 Nginx 服务器整体运行的配置指令。比如：`worker_processes 1`；worker_processes 值越大，可以支持的并发处理量就越多。
- **Events 块，**涉及的指令主要影响 Nginx 服务器与用户的网络连接。比如：`worker_connections 1024`；支持的最大连接数。
- **HTTP 块，**又包括 HTTP 全局块和 Server 块，是服务器配置中最频繁的部分，包括配置代理、缓存、日志定义等绝大多数功能。Server 块：配置虚拟主机的相关参数。Location 块：配置请求路由，以及各种页面的处理情况。

②配置文件

```json
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}   
```

### 3.2 配置示例

​		下面通过配置 Nginx 配置文件，实现正向代理、反向代理、负载均衡、Nginx 缓存、动静分离和高可用 Nginx 6 种功能，并对 Nginx 的原理作进一步的解析。

#### 3.2.1 java和tomcat安装及配置

①java和tomcat安装及配置

- 上传jdk和tomcat安装包到服务器
- 解压jdk及tomcat安装包到`/usr/local/`目录下，并对解压后的安装包更名

```bash
[root@localhost software]# tree
.
├── apache-tomcat-10.0.0-M3.tar.gz
└── jdk-8u161-linux-x64.tar.gz
[root@localhost software]# tar zxvf apache-tomcat-10.0.0-M3.tar.gz -C /usr/local/
···省略···
[root@localhost software]# tar zxvf jdk-8u161-linux-x64.tar.gz -C /usr/local/
···省略···
[root@localhost software]# cd /usr/local/
[root@localhost software]# mv apache-tomcat-10.0.0-M3/ tomcat
[root@localhost software]# mv jdk1.8.0_161/ jdk
[root@localhost local]# ls
bin  etc  games  include  jdk  lib  lib64  libexec  sbin  share  src  tomcat
```

②java环境变量配置

- 修改`/etc/profile`文件内容，命令：`vim /etc/profile`

```bash
# /etc/profile
export JAVA_HOME=/usr/local/jdk
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc
···省略···
```

- 初始化文件，使之立即生效

```bash
[root@localhost local]# source /etc/profile
```

- 测试jdk安装是否成功，如果能看到java version信息表示安装成功，否则失败。

```bash
[root@localhost local]# java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

③运行tomcat

- 启动tomcat服务

```bash
[root@localhost tomcat]# cd /usr/local/tomcat/
[root@localhost tomcat]# ./bin/startup.sh
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
[root@localhost tomcat]# ps -ef | grep tomcat
root      20906      1 11 00:43 pts/0    00:00:03 /usr/local/jdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat-Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
root      20947  20764  0 00:44 pts/0    00:00:00 grep --color=auto tomcat
```

- 测试tomcat

本地浏览器地址栏输入`http://192.168.5.158:8080/`，看到以下页面说明tomcat安装成功；否则，失败。

<img src=".\imgs\tomcat测试.png" style="zoom:75%;" />

#### 3.2.2 反向代理1

**实现效果**：在浏览器输入 `www.123.com` , 从 Nginx 服务器跳转到 Tomcat 主页面。

①启动tomcat服务

②通过修改本地 host 文件，将 `www.123.com` 映射到 192.168.5.158

<img src=".\imgs\反向代理1.png" style="zoom:75%;" />

配置完成之后，我们便可以通过 `www.123.com` 访问到 Tomcat 初始界面

③修改nginx配置文件，并重启nginx服务

​		在nginx配置文件中配置以下内容，命令`vim /etc/nginx/conf.d/default.conf`

<img src=".\imgs\反向代理2.png" style="zoom:75%;" />

​		重启nginx服务

```bash
[root@localhost ~]# nginx -s reload
```

④测试

在本地浏览器地址栏输入`http://www.123.com/`，看到以下页面说明tomcat安装成功；否则，失败。

<img src=".\imgs\反向代理3.png" style="zoom:75%;" />

**Location** 指令说明：

- **~：**表示 uri 包含正则表达式，且区分大小写。
- **~\*：**表示 uri 包含正则表达式，且不区分大小写。
- **=：**表示 uri 不含正则表达式，要求严格匹配。

#### 3.2.3 反向代理2

**实现效果**：使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中，同时设置nginx 监听端口为 9001。访问`http://192.168.5.158:9001/edu/`直接跳转到`192.168.5.158:8001`；访问`http://192.168.5.158:9001/vod/`直接跳转到`192.168.5.158:8002`

①准备两个tomcat服务

递归复制`/usr/local/`目录下的tomcat文件夹，得到另一个tomcat服务程序

```bash
[root@localhost local]# cp tomcat/ tomcat.bak -r
```

修改/usr/local/tomcat/conf/server.xml

②分别修改两个tomcat服务的服务端口

修改tomcat目录下的服务端口为8001

```bash
[root@localhost ~]# cd /usr/local/tomcat/conf/
[root@localhost conf]# vim server.xml
```

具体修改内容如下：

<img src=".\imgs\反向代理2_1.png" style="zoom:75%;" />

③修改tomcat.bak目录下的服务端口为8002

```bash
[root@localhost conf]# cd /usr/local/tomcat.bak/conf/
[root@localhost conf]# vim server.xml
```

具体修改内容如下：

<img src=".\imgs\反向代理2_2.png" style="zoom:75%;" />

<img src=".\imgs\反向代理2_3.png" style="zoom:75%;" />

④分别在两个tomcat服务的webapps目录下创建不同的文件

- 在`/usr/local/tomcat/webapps`目录下创建文件夹edu，同时edu目录下创建文件a.html

  ```bash
  [root@localhost webapps]# mkdir edu
  [root@localhost webapps]# cd edu
  ```

  创建文件a.html，命令`vim a.html`内容如下：

  ```html
  <h1>8001</h1>
  ```

- 在`/usr/local/tomcat.bak/webapps`目录下创建文件夹vod，同时vod目录下创建文件a.html

  ```bash
  [root@localhost webapps]# mkdir vod
  [root@localhost webapps]# cd vod
  ```

  创建文件a.html，命令`vim a.html`内容如下：

  ```html
  <h1>8002</h1>
  ```

- 找到nginx  配置文件，进行反向代理配置，命令`vim /etc/nginx/conf.d/default.conf`，具体配置内容如下：

  <img src=".\imgs\反向代理2_4.png" style="zoom:75%;" />

  然后，重启nginx服务

  ```bash
  [root@localhost ~]# nginx -s reload
  ```


⑤测试

- 启动两个tomcat服务


- 在本地浏览器地址栏输入`http://192.168.5.158:8001/edu/a.html`，看到以下页面


<img src=".\imgs\反向代理2_5.png" style="zoom:75%;" />

- 在本地浏览器地址栏输入`http://192.168.5.158:8001/vod/a.html`，看到以下页面


<img src=".\imgs\反向代理2_6.png" style="zoom:75%;" />

#### 3.2.4 负载均衡

**实现效果：**在浏览器地址栏输入 `http://192.168.5.158/edu/a.html` ，平均到 8001和 8002 端口中，实现负载均衡效果。

①准备两个tomcat服务，一个端口8001，另一个端口8002，具体操作同3.2.3节的①、②和③。

②在`/usr/local/tomcat.bak/webapps`目录下创建文件夹edu，同时edu目录下创建文件a.html

```bash
[root@localhost webapps]# mkdir edu
[root@localhost webapps]# cd edu
```

创建文件a.html，命令`vim a.html`内容如下：

```html
<h1>8001</h1>
```

③在 nginx 的配置文件`/etc/nginx/conf.d/default.conf`中进行负载均衡的配置

命令：`vim /etc/nginx/conf.d/default.conf`

文件内容如下：

```json
upstream myserver {   
  server 192.168.5.158:8001;
  server 192.168.5.159:8002;
}

server {
    listen       80;   #监听端口
    server_name  192.168.5.158;   #监听地址

    location  / {       
       root html;  #html目录
       index index.html index.htm;  #设置默认页
       proxy_pass  http://myserver;  #请求转向 myserver 定义的服务器列表      
    } 
}
```

③测试

- 启动nginx服务

  ```
  [root@localhost ~]# nginx -s reload
  ```

- 启动两个tomcat服务

  ```bash
  [root@localhost ~]# cd /usr/local/tomcat
  [root@localhost tomcat]# ./bin/startup.sh
  Using CATALINA_BASE:   /usr/local/tomcat
  Using CATALINA_HOME:   /usr/local/tomcat
  Using CATALINA_TMPDIR: /usr/local/tomcat/temp
  Using JRE_HOME:        /usr/local/jdk
  Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
  Tomcat started.
  [root@localhost tomcat]# cd /usr/local/tomcat.bak/
  [root@localhost tomcat.bak]# ./bin/startup.sh
  Using CATALINA_BASE:   /usr/local/tomcat.bak
  Using CATALINA_HOME:   /usr/local/tomcat.bak
  Using CATALINA_TMPDIR: /usr/local/tomcat.bak/temp
  Using JRE_HOME:        /usr/local/jdk
  Using CLASSPATH:       /usr/local/tomcat.bak/bin/bootstrap.jar:/usr/local/tomcat.bak/bin/tomcat-juli.jar
  Tomcat started.
  [root@localhost tomcat.bak]# ps -ef | grep tomcat
  root       7647      1 17 08:18 pts/1    00:00:04 /usr/local/jdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
  root       7694      1 56 08:18 pts/1    00:00:03 /usr/local/jdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat.bak/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat.bak/bin/bootstrap.jar:/usr/local/tomcat.bak/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat.bak -Dcatalina.home=/usr/local/tomcat.bak -Djava.io.tmpdir=/usr/local/tomcat.bak/temp org.apache.catalina.startup.Bootstrap start
  root       7734   7435  0 08:18 pts/1    00:00:00 grep --color=auto tomcat
  ```

- 在本地浏览器地址栏输入`http://192.168.5.158:8001/edu/a.html`，看到以下页面

  <img src=".\imgs\负载均衡1.png" style="zoom:75%;" />

- 在浏览器中多刷新几次可以看到

  <img src=".\imgs\负载均衡2.png" style="zoom:75%;" />

##### Nginx 分配服务器策略

**①轮询（默认）：**按请求的时间顺序依次逐一分配，如果服务器 down 掉，能自动剔除。

**②权重：**weight 越高，被分配的客户端越多，默认为 1。

比如：

```json
upstream myserver {   
	server 192.167.4.32:5000 weight=10;
	server 192.168.4.32:8080 weight=5;
}
```

**③IP：**按请求 IP 的 Hash 值分配，每个访客固定访问一个后端服务器。

比如：

```json
upstream myserver { 
    ip_hash;  
    server 192.167.4.32:5000;
    server 192.168.4.32:8080;
}
```

**④Fair：**按后端服务器的响应时间来分配，响应时间短的优先分配到请求。

比如：

```json
upstream myserver { 
    fair;  
    server 192.168.5.158:8001;
    server 192.168.5.158:8002;
}
```

#### 3.2.5 Nginx缓存

**实现效果：**在 3 天内，通过浏览器地址栏访问 `http://192.168.5.158/imgs/tmp.jpg`，不会从服务器抓取资源，3 天后（过期）则从服务器重新下载。

①在`/usr/local/tomcat/webapps/imgs/`目录下放一个文件tmp.jpg

②在 nginx 的配置文件`/etc/nginx/conf.d/default.conf`中进行缓存的配置

命令：`vim /etc/nginx/conf.d/default.conf`

文件内容如下：

```json
# server 区域下添加缓存配置
proxy_cache_path /tmp/nginx_proxy_cache levels=1 keys_zone=cache_one:512m inactive=60s max_size=1000m;

    # server 区域下添加缓存配置
    listen       80;
    server_name  www.123.com;
    
    # server 区域下添加缓存配置
    location / {
        proxy_pass http://localhost:8001;
        index index.html index.htm index.jsp;
    }

    # server 区域下添加缓存配置
    location ~ \.(gif|jpg|png|htm|html|css|js)(.*) {
         proxy_pass http://192.168.5.158:8001; # 如果没有缓存则转向请求
         proxy_redirect off;
         proxy_cache cache_one;
         proxy_cache_valid 200 1h;             # 对不同的 HTTP 状态码设置不同的缓存时间
         proxy_cache_valid 500 1d;
         proxy_cache_valid any 1m;
         expires 3d;
    }
```

③测试

- 启动tomcat

  ```bash
  [root@localhost tomcat]# cd
  [root@localhost ~]# cd /usr/local/tomcat
  [root@localhost tomcat]# ./bin/startup.sh
  Using CATALINA_BASE:   /usr/local/tomcat
  Using CATALINA_HOME:   /usr/local/tomcat
  Using CATALINA_TMPDIR: /usr/local/tomcat/temp
  Using JRE_HOME:        /usr/local/jdk
  Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
  Tomcat started.
  [root@localhost ~]# ps -ef | grep tomcat
  root      16664      1  4 17:52 pts/0    00:00:04 /usr/local/jdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat-Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
  root      16713  16407  0 17:54 pts/0    00:00:00 grep --color=auto tomcat
  ```

- 启动nginx

  ```bash
  [root@localhost ~]# nginx
  [root@localhost ~]# ps -ef | grep nginx
  root      16706      1  0 17:53 ?        00:00:00 nginx: master process nginx
  nginx     16707  16706  0 17:53 ?        00:00:00 nginx: worker process
  nginx     16708  16706  0 17:53 ?        00:00:00 nginx: cache manager process
  nginx     16709  16706  0 17:53 ?        00:00:00 nginx: cache loader process
  root      16711  16407  0 17:53 pts/0    00:00:00 grep --color=auto nginx
  ```

- 在本地浏览器地址栏输入`http://www.123.com/imgs/tmp.jpg`，将会看到以下页面

  <img src=".\imgs\nginx缓存1.png" style="zoom:75%;" />



- 查看服务器缓存文件

  ```bash
  [root@localhost ~]# ls -lh /tmp/nginx_proxy_cache/b/
  总用量 156K
  -rw-------. 1 nginx nginx 153K 4月   2 17:59 f09106f2af55d60077cba2b18a1fc53b
  [root@localhost ~]# ls -lh /usr/local/tomcat/webapps/imgs/
  总用量 152K
  -rw-r--r--. 1 root root 152K 4月   2 17:32 tmp.jpg
  ```

#### 3.2.6 动静分离

**实现效果：**通过浏览器地址栏访问 `http://192.168.5.158/image/tmp.jpg`访问到动态资源，通过浏览器地址栏访问 `http://192.168.5.158/www/a.html`访问到静态资源。

①在服务器根目录下创建data目录，然后在data目录下载创建`www`和`image`目录分别放置静态资源和动态资源。

最终的目录结构如下所示：

```bash
[root@localhost ~]# tree /data/
/data/
├── image
│   └── tmp.jpg
└── www
    └── a.html
```

②在 nginx 的配置文件`/etc/nginx/conf.d/default.conf`中进行配置

命令：`vim /etc/nginx/conf.d/default.conf`

```json
    # server 区域下添加缓存配置
    listen       80;
    server_name  localhost;

    # server 区域下添加缓存配置
    location /www/ {
        root /data/;
        index index.html index.htm;
    }

    # server 区域下添加缓存配置
    location /image/ {
        root /data/;
        autoindex on;           # 自动打开文件列表
    }
```

③测试

- 启动nginx

  ```bash
  [root@localhost ~]# nginx
  [root@localhost ~]# ps -ef | grep nginx
  root      16706      1  0 17:53 ?        00:00:00 nginx: master process nginx
  nginx     16707  16706  0 17:53 ?        00:00:00 nginx: worker process
  nginx     16708  16706  0 17:53 ?        00:00:00 nginx: cache manager process
  nginx     16709  16706  0 17:53 ?        00:00:00 nginx: cache loader process
  root      16711  16407  0 17:53 pts/0    00:00:00 grep --color=auto nginx
  ```

- 在本地浏览器地址栏输入`http://192.168.5.158/image/tmp.jpg`，将会看到以下页面

<img src=".\imgs\动静分离1.png" style="zoom:75%;" />

- 在本地浏览器地址栏输入`http://192.168.5.158/image/`，将会看到以下页面

<img src=".\imgs\动静分离2.png" style="zoom:75%;" />

- 在本地浏览器地址栏输入`http://192.168.5.158/www/a.html`，将会看到以下页面

<img src=".\imgs\动静分离3.png" style="zoom:75%;" />

#### 3.2.7 高可用性

**实现效果：**准备两台 Nginx 服务器，通过浏览器地址栏访问虚拟 IP 地址，把主服务器的 Nginx 停止，再次访问虚拟 IP 地址仍旧有效。

①在两台Nginx服务器上都安装keepalived

在两台Nginx服务器上分别执行下面的命令

```bash
[root@localhost ~]# yum install keepalived -y
···省略···
[root@localhost ~]# rpm -q -a keepalived
keepalived-1.3.5-16.el7.x86_64
```

②修改主备服务器 `/etc/keepalived/keepalived.conf` 配置文件（可直接替换），完成高可用主从配置

- Nginx主服务器keepalived配置文件

  ```bash
  global_defs {
      notification_email {
          acassen@firewall.loc
          failover@firewall.loc
          sysadmin@firewall.loc
      }
      notification_email_from_Alexandre.Cassen@firewall.loc
      smtp_server localhost
      smtp_connect_timeout 30
      router_id LVS_DEVEL  # 在 /etc/hosts 文件中配置，通过它能访问到我们的主机
  }
  
  vrrp_script_chk_http_port {
      script "/usr/local/src/nginx_check.sh"
  
      interval 2      # 检测脚本执行的时间间隔
  
      weight 2        # 权重每次加2
  }
  
  vrrp_instance VI_1 {
      interface ens33 # 网卡，需根据情况修改
      state MASTER    # 备份服务器上将 MASTER 改为 BACKUP
      virtual_router_id 51 # 主备机的 virtual_router_id 必须相同
      priority 100   # 主备机取不同的优先级，主机值较大，备份机值较小
      advert_int 1  # 每隔多长时间（默认1s）发送一次心跳，检测服务器是否还活着
      authentication {
        auth_type PASS
        auth_pass 1111
      }
      virtual_ipaddress {
          192.168.5.100 # VRRP H 虚拟地址，可以绑定多个
      }
  }
  ```

- Nginx备份服务器keepalived配置文件

  ```bash
  global_defs {
      notification_email {
          acassen@firewall.loc
          failover@firewall.loc
          sysadmin@firewall.loc
      }
      notification_email_from_Alexandre.Cassen@firewall.loc
      smtp_server localhost
      smtp_connect_timeout 30
      router_id LVS_DEVEL  # 在 /etc/hosts 文件中配置，通过它能访问到我们的主机
  }
  
  vrrp_script_chk_http_port {
      script "/usr/local/src/nginx_check.sh"
  
      interval 2      # 检测脚本执行的时间间隔
  
      weight 2        # 权重每次加2
  }
  
  vrrp_instance VI_1 {
      interface ens33 # 网卡，需根据情况修改
      state BACKUP    # 备份服务器上将 MASTER 改为 BACKUP
      virtual_router_id 51 # 主备机的 virtual_router_id 必须相同
      priority 90   # 主备机取不同的优先级，主机值较大，备份机值较小
      advert_int 1  # 每隔多长时间（默认1s）发送一次心跳，检测服务器是否还活着
      authentication {
        auth_type PASS
        auth_pass 1111
      }
      virtual_ipaddress {
          192.168.5.100 # VRRP H 虚拟地址，可以绑定多个
      }
  }
  ```

- 文件`/etc/keepalived/keepalived.conf`字段说明

  **router_id：**在 /etc/hosts 文件中配置，通过它能访问到我们的主机。

  ```
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    127.0.0.1   LVS_DEVEL   
  ```

  **interval：**设置脚本执行的间隔时间。

  **weight：**当脚本执行失败即 Keepalived 或 Nginx 挂掉时，权重增加的值（可为负数）。

  **interface：**输入 `ip a` 命令查看当前的网卡名是什么。

  ```
  [root@localhost ~]# ip a
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether 00:0c:29:96:ae:a3 brd ff:ff:ff:ff:ff:ff
      inet 192.168.5.158/24 brd 192.168.5.255 scope global noprefixroute dynamic ens33
         valid_lft 1176sec preferred_lft 1176sec
      inet6 fe80::52f:55f9:4d3c:9847/64 scope link noprefixroute
         valid_lft forever preferred_lft forever
  ```

③在两台服务器的/`usr/local/src` 目录下添加检测脚本 `nginx_check.sh`

在两台服务器的/`usr/local/src` 目录下都需要添加nginx_chekc.sh，其内容如下：

```sh
#!/bin/bash
A=`ps -C nginx -no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/sbin/nginx # 这是nginx命令所在路径，可能需要自行配置
    sleep 2
    if [ ps -C nginx -no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
```

④测试

- 首先需要将两台服务器的nginx配置文件`/etc/nginx/conf.d/default.conf`恢复初始状态

- 启动主服务器的keepalived和nginx

  ```bash
  [root@localhost ~]# systemctl start keepalived
  [root@localhost ~]# nginx
  [root@localhost ~]# ps -ef | grep keepalived
  root       7271      1  0 22:45 ?        00:00:00 /usr/sbin/keepalived -D
  root       7272   7271  0 22:45 ?        00:00:00 /usr/sbin/keepalived -D
  root       7273   7271  0 22:45 ?        00:00:00 /usr/sbin/keepalived -D
  root       7279   7129  0 22:45 pts/0    00:00:00 grep --color=auto keepalived
  [root@localhost ~]# ps -ef | grep nginx
  root       7276      1  0 22:45 ?        00:00:00 nginx: master process nginx
  nginx      7277   7276  0 22:45 ?        00:00:00 nginx: worker process
  root       7281   7129  0 22:45 pts/0    00:00:00 grep --color=auto nginx
  ```

- 启动备份服务器的keepalived和nginx

  ```bash
  [root@localhost ~]# systemctl start keepalived
  [root@localhost ~]# nginx
  [root@localhost ~]# ps -ef | grep keepalived
  root       7255      1  0 22:46 ?        00:00:00 /usr/sbin/keepalived -D
  root       7256   7255  0 22:46 ?        00:00:00 /usr/sbin/keepalived -D
  root       7257   7255  0 22:46 ?        00:00:00 /usr/sbin/keepalived -D
  root       7264   7144  0 22:46 pts/0    00:00:00 grep --color=auto keepalived
  [root@localhost ~]# ps -ef | grep nginx
  root       7259      1  0 22:46 ?        00:00:00 nginx: master process nginx
  nginx      7260   7259  0 22:46 ?        00:00:00 nginx: worker process
  root       7266   7144  0 22:46 pts/0    00:00:00 grep --color=auto nginx
  ```

- 在本地浏览器地址访问指定页面

  - 在浏览器地址栏输入`http://192.168.5.100/`，将会看到以下页面

    <img src=".\imgs\高可用性1.png" style="zoom:75%;" />

  - 停止主服务器的keepalived和nginx服务

    ```bash
    [root@localhost ~]# systemctl stop keepalived
    [root@localhost ~]# nginx -s stop
    [root@localhost ~]# ps -ef | grep keepalived
    root       7357   7129  0 23:15 pts/0    00:00:00 grep --color=auto keepalived
    [root@localhost ~]# ps -ef | grep nginx
    root       7359   7129  0 23:15 pts/0    00:00:00 grep --color=auto nginx
    ```

  - 在浏览器中刷新页面，继续访问`http://192.168.5.100/`，还可以看到以下页面

    <img src=".\imgs\高可用性1.png" style="zoom:75%;" />

## 4 Nginx原理解析

①master和worker

<img src=".\imgs\640(6).png" style="zoom:75%;" />

​		Nginx 启动之后，在 Linux 系统中有两个进程，一个为 Master，一个为 Worker。Master 作为管理员不参与任何工作，只负责给多个 Worker 分配不同的任务（Worker 一般有多个）。

```bash
[root@localhost ~]# ps -ef | grep nginx
root       7377      1  0 23:20 ?        00:00:00 nginx: master process nginx
nginx      7378   7377  0 23:20 ?        00:00:00 nginx: worker process
root       7401   7129  0 23:20 pts/0    00:00:00 grep --color=auto nginx
```

②worker工作原理

​        客户端发送一个请求首先要经过 Master，管理员收到请求后会将请求通知给 Worker。多个 Worker 以争抢的机制来抢夺任务，得到任务的 Worker 会将请求经由 Tomcat 等做请求转发、反向代理、访问数据库等（Nginx 本身是不直接支持 Java 的）。

<img src=".\imgs\640(7).png" style="zoom:75%;" />

③一个 Master 和多个 Worker 的好处？
- 可以使用 nginx -s reload 进行热部署。
- 每个 Worker 是独立的进程，如果其中一个 Worker 出现问题，其他 Worker 是独立运行的，会继续争抢任务，实现客户端的请求过程，而不会造成服务中断。

④设置多少个 Worker 合适？

- Nginx 和 Redis 类似，都采用了 IO 多路复用机制，每个 Worker 都是一个独立的进程，每个进程里只有一个主线程。通过异步非阻塞的方式来处理请求，每个 Worker 的线程可以把一个 CPU 的性能发挥到极致，因此，**Worker 数和服务器的 CPU 数相等是最为适宜的**。

⑤思考：

1. 发送一个请求，会占用 Worker 几个连接数？

2. 有一个 Master 和 4 个 Worker，每个 Worker 支持的最大连接数为 1024，该系统支持的最大并发数是多少？

   1. 答案：2或者4

   2. 答案：

      普通的静态访问最大并发数是： worker_connections * worker_processes /2 

      而如果是 HTTP  作  为反向代理来说，最大并发数量应该是 worker_connections * worker_processes/4