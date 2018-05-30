---
layout: post
title: jenkins自动部署spring-boot项目
category: 技术
tags: PHP
description: jenkins自动部署spring-boot项目
---


一、jenkins安装
1、使用的软件版本
gitlab10.1,maven3.5.3,git1.8.3.1,jenkins2.107.3,jdk1.8.0_121,centos7.3
2、安装maven
（1）、下载maven
从官网http://maven.apache.org/download.cgi，下载maven安装包，并上传到服务器指定目录。
（2）、安装maven
解压：tar xf apache-maven-3.5.3-bin.tar.gz
移动：mv apache-maven-3.5.3-bin /usr/local/maven3
（3）、配置maven
修改环境变量， 在/etc/profile中添加以下几行
MAVEN_HOME=/usr/local/maven3
export MAVEN_HOMEexport PATH=${PATH}:${MAVEN_HOME}/bin
执行source /etc/profile使环境变量生效。
验证 最后运行mvn -v验证maven是否安装成功
3、关闭防火墙
执行命令如下：systemctl stop firewalld.service，查看防火墙状态：service iptables status
4、安装git
命令如下：yum install git
查看git是否安装成功：git --version
5、安装jenkins
（1）、安装命令如下
#sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
#sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
#yum install jenkins
（2）、安装文件信息
jenkins.war存储目录为/usr/lib/jenkins/，jenkins初始密码存储文件：	/var/lib/jenkins/secrets/initialAdminPassword
（3）、修改jenkins的jdk路径
打开文件vim /etc/init.d/jenkins，在candidates中添加服务器的jdk目录，如：/usr/local/jdk1.8/bin/java
（4）、修改jenkins的端口
打开jenkins配置文件/etc/sysconfig/jenkins，修改其中的端口，如8080修改位9080
（5）、jenkins启动
命令如下：java -jar jenkins.war --ajp13Port=-1 --httpPort=9080
二、jenkins配置
1、配置启动jenkins
Jenkins 就启动成功了！它的war包自带Jetty服务器
第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。注意控制台输出的口令，复制下来，然后在浏览器输入密码：
INFO: 
**************************************************************************************************************************************************************************************
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
0cca37389e6540c08ce6e4c96f46da0f
This may also be found at: /root/.jenkins/secrets/initialAdminPassword
***************************************************************************************************************************************************************************************
访问 浏览器访问：http://172.28.50.92:9080/

输入：0cca37389e6540c08ce6e4c96f46da0f
进入用户自定义插件界面，建议选择安装官方推荐插件，因为安装后自己也得安装:

接下来是进入插件安装进度界面:

插件一次可能不会完全安装成功，可以点击Retry再次安装。直到全部安装成功

等待一段时间之后，插件安装完成，配置用户名密码:

输入：admin/admin
系统管理-》全局工具配置 jdk路径，

2、插件安装和配置
有很多插件都是选择的默认的安装的，所以现在需要我们安装的插件不多，Git plugin和Maven Integration plugin，publish over SSH。
插件安装：系统管理 > 插件管理 > 可选插件,勾选需要安装的插件，点击直接安装或者下载重启后安装

配置全局变量
系统管理 > 全局工具配置
JDK
配置本地JDK的路径，去掉勾选自动安装

Maven
配置本地maven的路径，去掉勾选自动安装

其它内容可以根据自己的情况选择安装。
配置 SSH免登陆
ssh的配置可使用密钥，也可以使用密码，这里我们使用密钥来配置，在配置之前先配置好jenkins服务器和应用服务器的密钥认证 jenkins服务器上生成密钥对，使用ssh-keygen -t rsa命令
输入下面命令 一直回车，一个矩形图形出现就说明成功，在~/.ssh/下会有私钥id_rsa和公钥id_rsa.pub
ssh-keygen -t rsa
将jenkins服务器的公钥id_rsa.pub中的内容复制到应用服务器 的~/.ssh/下的 authorized_keys文件
ssh-copy-id -i id_rsa.pub 192.168.0.xx
chmod 644 authorized_keys
在应用服务器上重启ssh服务，service sshd restart现在jenkins服务器可免密码直接登陆应用服务器
之后在用ssh B尝试能否免密登录B服务器，如果还是提示需要输入密码，则有以下原因
a. 非root账户可能不支持ssh公钥认证（看服务器是否有限制）
b. 传过来的公钥文件权限不够，可以给这个文件授权下 chmod 644 authorized_keys
c. 使用root账户执行ssh-copy-id -i ~/.ssh/id_rsa.pub 这个指令的时候如果需要输入密码则要配置sshd_config
vi /etc/ssh/sshd_config#内容
PermitRootLogin no
修改完后要重启sshd服务
service sshd restart
最后，如果可以SSH IP 免密登录成功说明SSH公钥认证成功。
3、Push SSH配置
系统管理 > 系统设置
选择 Publish over SSH

Passphrase  sshserver服务器的密码,填写了改密码，则Key和Path to key不用填写
Path to key 写上生成的ssh路径：/root/.ssh/id_rsa
下面的SSH Servers是重点
Name 随意起名代表这个服务，待会要根据它来选择 
Hostname 配置应用服务器的地址 
Username 配置linux登陆用户名 
Remote Directory 不填，默认是服务器的顶级目录”/”
点击下方增加可以添加多个应用服务器的地址

三、jenkins项目配置
（1）、创建maven项目

（2）、配置项目信息和构建保存最大个数

（3）、配置源码管理

（4）、配置构建触发器

（5）配置Pre Steps

（6）配置Post Steps

其中：Either Source files, Exec command or both must be supplied，不用管，没有影响。
（7）点击应用，保存
（8）进入到项目页

（9）选择立即构建，构建历史会出现如下信息：

（10）点击#27，选择控制台输出：

出现如下信息，则文件发送成功，构建成功。

（11）访问http://172.28.50.91:9081/，则出现对应界面。

四、jenkins配置自动部署
1、安装Gitlab Hook Plugin插件

#系统管理-管理插件-可选插件-Gitlab Hook Plugin和Build Authorization Token Root Plugin

2、在jenkins服务器生成随机token：
# openssl rand -hex 12
0f2a47c861133916d2e299e3
 
3、创建项目触发器：
#项目-配置-构建触发器：
 http://172.28.50.92:9080/project/demo

4：配置gitlab：
(1)：在git项目配置界面设置链接和token
登录gitlab,在这个项目下找到配置的地方:

然后选择Push events，即推送时构建，保存。
（2）、测试：

报错：hook executed successfully but returned http 403
因为jenkins的安全策略设置不能访问，可以去除jenkins的安全策略，如下：

（3）、测试，看到显示200表示成功
修改代码，推送最新代码到gitlab，然后查看jenkins是否可以编译最新代码，如可编译，则查看项目是否可以访问和修改的内容是否显示。


五、遇到的问题：
1、jar包传不到指定服务器
jenkins全局Remote Directory改为/，Post Steps配置时工作目录改为target目录：target/demo-0.0.1-SNAPSHOT.jar，Remove prefix改为：target/，
2、Remote directory 改为jar包所需在的目录。
3、Gitlab提交不能自动部署
去除jenkins的登录安全检查。
4、Gitlab10.1的Webhooks改为了Integrations。

参考文档：http://www.mooooc.com/springboot/2017/11/11/springboot-jenkins.html和https://www.jianshu.com/p/d4f2953f3ce0和https://www.cnblogs.com/reblue520/p/7146638.html，
https://blog.csdn.net/minicto/article/details/73614124，https://www.cnblogs.com/reblue520/p/7130914.html

