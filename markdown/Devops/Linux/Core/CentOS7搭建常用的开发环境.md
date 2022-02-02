# CentOS7搭建常用的开发环境

本篇讲解如何在CentOS7上搭建常见的Java、Python、NodeJS、Docker的开发环境。

## 安装和配置JDK
先查查看系统上面是否有其他的旧版本，有的话就卸载掉：

```bash
sudo rpm -qa | grep jdk
jdk-1.7.0_45-fcs.x86_64
sudo rpm -e jdk-1.7.0_45
```

从Oracle官网下载最新的JDK 11压缩包：<https://www.oracle.com/java/technologies/downloads/#java11>

选择`jdk-11.0.14_linux-x64_bin.tar.gz`进行下载，复制到目录`/usr/local/`中。

解压安装包，重命名为jdk：
```
tar -zxf jdk-11.0.14_linux-x64_bin.tar.gz
mv jdk-11.0.14 jdk
```

配置环境变量，`vim /etc/profile`，在文件尾添加：
``` bash
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

如果想要立即生效，执行`source /etc/profile`

## 安装和配置Maven
首先在Apache Maven 官网下载Maven的安装包：<https://maven.apache.org/download.cgi>

下载文件`apache-maven-3.8.4-bin.tar.gz`，复制到目录`/usr/local/`中。

然后解压下载的 Maven 安装包，将文件夹重命名为maven：
```
tar -zxvf apache-maven-3.8.4-bin.tar.gz
mv apache-maven-3.8.4 maven
```

添加Maven的环境变量，`vim /etc/profile`，在最后添加：
```
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH
```

想要理解生效则执行：`source /etc/profile`。

然后执行`mvn -v`查看是否设置成功。

打开maven配置文件，`vim /usr/local/maven/conf/settings.xml`，修改仓库为阿里云的仓库：
``` xml
<mirrors>
   <mirror>
     <id>alimaven</id>
     <name>aliyun maven</name>
     <url>https://maven.aliyun.com/nexus/content/groups/public/</url>
     <mirrorOf>central</mirrorOf>
   </mirror>
</mirrors>
```

配置Maven本地仓库目录，首先创建一个repository：
```
mkdir /usr/local/maven/repository
```

然后修改Maven的配置settings.xml，指定仓库地址为该目录：
```
<localRepository>/usr/local/maven/repository</localRepository>
```

## 安装和配置git
使用yum安装的版本号太低了，推荐使用源码安装git。

安装所需软件包：
```
yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel 
yum install -y gcc-c++ perl-ExtUtils-MakeMaker
```

然后要卸载旧版本git（因为在安装依赖过程中，yum工具自动安装了旧版本的git）
```
yum remove git
```

访问git的官网，下载最新版本的压缩包：<https://github.com/git/git/tags>

下载后复制到目录`/root/`中，并进行解压缩，然后进行make编译并安装：
```
tar zxf git-2.35.1.tar.gz
cd git-2.35.1
make prefix=/usr/local/git all
make prefix=/usr/local/git install
```

最后添加添加环境变量`vim /etc/profile`，在最后添加如下
```
export PATH=/usr/local/git/bin:$PATH
```
想要理解生效则执行：`source /etc/profile`。
然后查看`git --version`。

安装完成，还要简单配置下全局设置:
``` bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
# 彩色的git输出
git config --global color.ui true
# 显示历史记录时，每个提交的信息只显示一行
git config --global format.pretty oneline
```

## 安装和配置Python3
通过yum安装的python3版本太低，推荐使用源码安装Python3。

安装编译相关工具
```
yum -y groupinstall "Development tools"
yum install gcc openssl-devel bzip2-devel libffi-devel -y
```

先从官网下载最新源码包：<https://www.python.org/ftp/python/3.10.2/Python-3.10.2.tgz>

解压缩、make编译和安装：
```
tar -zxf Python-3.10.2.tgz
cd Python-3.10.2
./configure prefix=/usr/local/python3
make && make install
```

创建python3和pip3的链接：
```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

验证是否成功：
```
python3 --version
pip3 --version
```

配置pip的豆瓣源，创建一个配置文件，linux用户将它命名为pip.conf,
windows用户将它命名为pip.ini，文件中写如下内容：
```
[global]
timeout = 60
index-url = https://pypi.doubanio.com/simple
```

linux下指定位置为`$HOME/.pip/pip.conf`，windows下指定位置为`%HOME%\pip\pip.ini`。
如果你使用了virtualenv, 那么配置文件应该放置在virtualenv生成的文件夹中。

## 安装和配置NodeJS

Node.js的当前LTS版本是16.x版本。 如果要安装版本14，只需在以下命令中将setup_16.x更改为setup_14.x。
运行以下curl命令，将NodeSource yum存储库添加到您的系统中：
```
curl -fsSL https://rpm.nodesource.com/setup_16.x | sudo bash -
```

启用NodeSource存储库后，通过键入以下内容安装Node.js和npm：
```
yum install -y nodejs
```
当系统提示您导入存储库GPG密钥时，键入y，然后按Enter。

验证Node.js和npm的安装：
```
node --version
npm --version
```

配置NPM的国内淘宝源：

持久使用
```
npm config set registry https://registry.npm.taobao.org
```

查看npm源地址
```
npm config get registry
```

## 安装和配置Docker

```
# 卸载旧版本（如果之前安装过的话）
yum remove docker  docker-common docker-selinux docker-engine
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
```

编辑`/etc/docker/daemon.json`，如果没有就创建一个：

```
sudo mkdir /etc/docker
vim /etc/docker/daemon.json
```

写入如下内容：

```
{
  "storage-driver": "devicemapper"
}
```

启动docker并查看状态：

```
sudo systemctl start docker
sudo systemctl status docker
```

**镜像下载加速**

由于 Docker Hub 的服务器在国外，下载镜像会比较慢。幸好 DaoCloud 为我们提供了免费的国内镜像服务。

下面介绍如果使用镜像服务。在`daocloud.io` 免费注册一个用户，可直接用GitHub用户注册。进入加速器页面: <https://www.daocloud.io/mirror>

对于linux平台，下面有一行脚本：
```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

我测试的时候，这个脚本会在`/etc/docker/daemon.json`中写入：
```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"],}
```

然后重启docker，体验飞一般的感觉：
```
sudo systemctl restart docker.service
sudo systemctl enable docker.service
```

**安装Compose**

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：<https://github.com/docker/compose/releases>

当前下载到的最新版本为v2.2.3，文件名为`docker-compose-linux-x86_64`，
将其复制到CentOS7的主机的`/usr/local/bin/docker-compose`。

将可执行权限应用于二进制文件：
```bash
chmod +x /usr/local/bin/docker-compose
```

创建软链：
```bash
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

如果想卸载docker-compose，只需要把二进制文件`/usr/local/bin/docker-compose`删除即可，非常简单。

