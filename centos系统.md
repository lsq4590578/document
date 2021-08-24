[TOC]

# 1.0.0centos 安装

如何安装一个proxmox系统的服务器，直接通过U盘烧制一个proxmox的系统。使用UltraISO将下载的[proxmox.iso](https://www.proxmox.com/en/downloads/item/proxmox-ve-7-0-iso-installer) 下载so镜像烧制到U盘中，通过系统启动安装，设置域名及端口号，默认端口号为8006。通过IP地址访问:http://192.168.1.95:8006

## 1.1.0 proxmox创建虚拟机。

### 1.1.1 上传镜像

​		我们公司的镜像放在nfs11这台的机子上，可以去下载对应的镜像，然后选择nfs11这个点击右侧的upload按钮，上传镜像

![image-20210611112358650](D:\技术文档\proxmox镜像.png)

### 1.1.2创建vm

- 点击对应的pve17的机子，然后创建creat vm,然后选择对应的ISO镜像，这边选择的是centos7.0的

![image-20210611113356356](D:\技术文档\centos系统\创建vm.png)

- 设置系统系统的Disk size 为120G
- 设置CPU核数 10核 
- 设置内存为10G
- 网络默认bridge vmbr0
- 设置完成后进入安装界面，设置了用户名的密码，选择是否需要分区

## 1.2.0 配置centos

### 1.2.1配置镜像下载源

​         扫码是yum仓库，yum仓库就是使用yum命令下载软件的镜像地址。

```bash
vim /etc/yum.repos.d/CentOS-Base.repo

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra #把这行注释掉
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/  #把mirror.centos.org改成mirrors.tuna.tsinghua.edu.cn
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra  #把这行注释掉
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/ #把mirror.centos.org改成mirrors.tuna.tsinghua.edu.cn
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra #把这行注释掉
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/ #把mirror.centos.org改成mirrors.tuna.tsinghua.edu.cn
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra  #把这行注释掉
baseurl=http://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/ #把mirror.centos.org改成mirrors.tuna.tsinghua.edu.cn
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

```

### 1.2.2设置静态网络

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0


TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static #设置静态IP
DEFROUTE=yes 
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=e9b0c370-ff28-4de3-95fc-67e91425e66b
DEVICE=eth0
ONBOOT=yes  #开启重启
IPADDR=192.168.1.24 #设置IP地址
GATEWAY=192.168.1.1 #网关
NETMASK=255.255.255.0 #子网掩码 
DNS1=192.168.1.1 #DNS解析
重启网络，或者reboot
```



### 1.2.3安装需要的内容

```
yum update
yum install wget
yum isntall net-tools #ifconfig才可以使用
yum install vim #常用的编辑工具
yum install python #安装python3.6.6 并且设置软连接，默认访问python3的地址
```



# 2.0.0centos部署JDK

官网下载一个jdk的安装包

```
 wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie;" http://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.tar.gz
 
```



# 3.0.0centos 部署Jenkins

## 3.1 yum方式安装

### 3.1.1 yum安装jenkins

使用这个需要配置jdk的安装路径，否则无法启动成功

```
yum install jenkins
#配置jenkins端口号
vim /etc/sysconfig/jenkins
修改jenkins JENKINS_PORT="8080" 可以修改成其他不冲突的端口号
#查看日志：/var/log/jenkins

```

### 3.1.2yum卸载jenkins

```bash
rpm -e jenkins #删除安装包
find / -iname jenkins | xargs -n 1000 rm -rf #删除安装的所有文件和目录
```

## 3.2 tomcat方式部署jenkins

###    3.2.1 tomcat 安装

- 创建指定目录mkdir tomcat
- 下载tomcat安装包: apache-tomcat-8.5.34.tar,使用wget也ok，直接下载到windows在用rz 命令上传也OK

```bash
tar -vxf apache-tomcat-8.5.34.tar #解压文件
cp /data/data/tomcat/apache-tomcat-8.5.34/bin
./startup.sh #启动tomcat
#验证tomcat
http://192.168.1.24:8080 
```

### 3.2.2 tomcat部署jenkins

此处要注意的是最好一步步确定没有问题再来操作下一步，比如tomcat下载好后，直接启动tomcat，试看看是否可以访问http://192.168.1.24:8080是否可以访问到tomcat，如果可以证明tomcat服务启动，无法访问再查找apache-tomcat-8.5.34/logs日志查看相关出错问题。

```bash
cd /data
mkdir jenkins
cd jenkins
wget http://mirrors.jenkins-ci.org/war/latest/jenkins.war
cp /data/data/tomcat/apache-tomcat-8.5.34/bin
cd jenkins/jenkins.war /data/tomcat/apache-tomcat-8.5.34/bin
./startup.sh #启动jenkins
```

### 3.2.3 配置jenkins的存储位置

注意：此处如果没有配置默认是在/root/.jenkins下面，过往的经验告诉我们这个是一个非常错误的教训，因为在root的磁盘分配的不够类似于我们的c盘。把jenkins的东西放到c盘后，后续东西越来越多导致需要迁移时，会出现各种各样的问题，所以养成不用默认安装的方式，给jenkins配置指定的空间

```bash
vim /etc/profile
#写入
JENKINS_HOME=/data/jenkins
export JENKINS_HOME
```



### 3.2.4 访问jenkins

```bash
http://192.168.1.24:8080/jenkins
```

### 3.2.5 出错解决

当一切都准备OK了，但是发现访问http://192.168.1.24:8080/jenkins的时候出现无法访问的问题

提供一下我当时的解决问题的思路

#### 3.2.5.1网络

- 为什么无法访问，访问服务的网络有问题吗？验证过百度可以访问，所以访问浏览器的这台客户端网络没有问题

- 那服务端的网络有问题吗？使用curl http://www.baidu.com返回的是baidu的html，服务端的网络也没有问题

- 这两台服务器的网络是否互通？客户端IP:192.168.1.91 服务器IP：192.168.1.24，分别在自己的自己ping对方，看是否网络是否可以访问，验证后，网络也是互通的。

  结合上面几个验证，说明网络没有问题，排除网络问题

  

#### 3.2.5.2 tomcat服务问题

-  这个是因为启动tomcat的时候根本没有测试，所以不知道tomcat启动是否有问题，所以在服务端测试一下

  ```
  curl http://192.168.1.24:8080 #看返回的是否带有tomcat相关内容，如果是，证明服务器启动正常
  ```

- 服务端验证jenkins是的访问地址是否有问题

  ```
  curl http://192.168.1.24:8080/jenkins #查看jenkins的访问页面是否可以请求到，如果可以服务器端没有问题
  ```



分析原因：

网络没有问题，服务端的服务器也没有问题，是什么导致这两个不能通信的？

防火墙

#### 3.2.5.3验证猜想

猜想是可能因为防火墙导致这客户端和服务端不能通信，那么就尝试把防火墙先关闭，然后访问看看

```
systemctl status firewalld  #查看防火墙状态，active 或者是running则证明启动了
systemctl stop firewalld #关闭防火墙
```

使用浏览器再次访问http://192.168.1.24:8080/jenkins，神奇般的启动了。



## 3.3 JENKINS的配置

## 	3.3.1 配置Plugin Manager的代理

进入Manage Jenkins -Manager Plugins - Advanced ：Update Site URL：https://updates.jenkins.io/update-center.json替换成如下的URL

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

## 	3.3.2 常用插件安装

# 4.0.0 安装git

## 4.1.0安装

```bash
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
tar -zvxf git-2.9.5.tar.gz
cd  git-2.9.5
./configure
make & make install 

```

## 4.2.0安装遇到的问题

### 4.2.1 git安装编译遇到的问题

```bash
usr/bin/perl Makefile.PL PREFIX='/usr/local/git' '' --localedir='/usr/local/git/share/locale'
Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at Makefile.PL line 3.
BEGIN failed--compilation aborted at Makefile.PL line 3.
make[1]: *** [perl.mak] Error 2
make: *** [perl/perl.mak] Error 2 
 

解决方法，执行：
yum install perl-ExtUtils-MakeMaker package

然后重新make & make install 
```

配置git环境变量

```BASH
vim /etc/profile 
export PATH=$PATH:/usr/local/bin/git
export PATH

#测试 
git --version
```

### 4.2.2 git使用过程中遇到问题

遇到的问题以及如何解决的

```
[root@localhost workspace]# git clone http://gitlab.klxuexi.org/test/erp_e2e.git
正克隆到 'erp_e2e'...
fatal: Unable to find remote helper for 'http'
```

- 解决办法：

```bash
yum install curl-devel
yum -y install git-core gitk git-gui 

重新进入重新编译和安装
cd /data/git/git-2.9.4
./configure --prefix=/usr/local/bin/
make && make install

```

以上操作未解决此问题，导致find remote helper for 'http'找不到是因为git-core包中的git-remote-http没有映射到安装目录

```bash
whereis git-core
#git-core: /usr/libexec/git-core /usr/share/git-core

#对创建软连接
 ln -s /usr/libexec/git-core/git-remote-http /usr/local/bin
 #在执行git clone 
 git clone http://gitlab.klxuexi.org/test/erp_e2e.git
 
 #顺利解决以上问题
```

# 5.0.0 Python 安装

## 5.1.0下载安装

此处需要注意的点是centos系统默认安装python2.7，所以需要额外安装python3以及pip3需要设置软连接

```bash
wget  https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tgz
tar -zvxf Python-3.6.6.tgz 
cd Python-3.6.6
./configure --prefix=/usr/local/
#如果有出错，则重新安装
 yum install -y gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
 #再次重新编译和安装
 ./configure --prefix=/usr/local/
  make && make install
```

## 5.2.0配置软链接

```
#设置软链接
ln -s /usr/local/bin/python3.6 /usr/local/bin   #python3为当前使用
ln -s /usr/local/pip3  /usr/local/bin/pip    #设置pip3为当前使用
```

## 5.3.0 虚拟环境

### 5.3.1安装虚拟环境

```bash
pip3 install virtualenv
pip3 install virtualenvwrapper

#查找：
find / -name virtualenvwrapper.sh
#或者
whereis virtualenvwrapper.sh

vim /root/.bashrc
export WORKON_HOME=/home/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.6
source /usr/local/bin/virtualenvwrapper.sh
```

### 5.3.2虚拟环境操作

```bash
mkvirtualenv #虚拟环境名
workon #虚拟环境名
deactivate #禁用虚拟环境名
workon+enter #查看虚拟环境列表
```

# 6.0.0 Docker 安装

# 7.0.0 NGNIX WEB服务器安装使用

## 7.1.0安装

```
cd /data/tools
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar -zvxf nginx-1.20.1.tar.gz

./confirgura \
--prefix=/data/nginx \
--sbin-path=/data/nginx/nginx \
--conf-path=/data/nginx/conf 

make && make install
```

## 7.2.0配置nginx.conf和项目静态文件配置

```bash
cd /data/nginx/conf
vim nginx.conf
在http{include conf.d/www/*.conf} #用于指向对应项目的nginx的配置文件
cd /data/nginx/conf/conf.d/www
touch hs.conf


#在浏览器中输入http://hitools.klxuexi.net-》通过服务器50的nginx的nginx.conf中的include文件找，看到#at.conf配置，转发出来到192.168.1.24的服务器nginx.conf中找是否有server_name等于这个的，没有则继续找#include对应的.conf文件中是否有，如果有直接找到include下对应的root 目录下的index.html文件显示出来。（前端页面配置）

#at.conf
server {
	server_name hitools.klxuexi.net;
    location / {
		proxy_pass http://192.168.1.24;
	}
}
server {
    server_name hitools.test.klxuexi.net;
    location / {
        proxy_pass http://192.168.1.39;
    }
}


server {
    listen 80;
    server_name hitools.klxuexi.net;
    root html/hitoolsserver;
    index index.html;
}

```

