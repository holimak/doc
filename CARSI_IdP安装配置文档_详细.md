<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Shibboleth IdP 安装配置文档 (详细)](#shibboleth-idp-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E6%96%87%E6%A1%A3-%E8%AF%A6%E7%BB%86)
  - [目标](#%E7%9B%AE%E6%A0%87)
  - [安装准备](#%E5%AE%89%E8%A3%85%E5%87%86%E5%A4%87)
  - [安装步骤](#%E5%AE%89%E8%A3%85%E6%AD%A5%E9%AA%A4)
    - [1. 安装前准备](#1-%E5%AE%89%E8%A3%85%E5%89%8D%E5%87%86%E5%A4%87)
      - [1.1 修改本机hostname](#11-%E4%BF%AE%E6%94%B9%E6%9C%AC%E6%9C%BAhostname)
      - [1.2 关闭selinux](#12-%E5%85%B3%E9%97%ADselinux)
      - [1.3 开放本机端口](#13-%E5%BC%80%E6%94%BE%E6%9C%AC%E6%9C%BA%E7%AB%AF%E5%8F%A3)
      - [1.4 设置时间同步](#14-%E8%AE%BE%E7%BD%AE%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5)
    - [2. 安装IdP运行环境](#2-%E5%AE%89%E8%A3%85idp%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83)
    - [3. 安装&本地配置IdP](#3-%E5%AE%89%E8%A3%85%E6%9C%AC%E5%9C%B0%E9%85%8D%E7%BD%AEidp)
      - [3.1 安装IdP](#31-%E5%AE%89%E8%A3%85idp)
      - [3.2 配置apache和tomcat](#32-%E9%85%8D%E7%BD%AEapache%E5%92%8Ctomcat)
      - [3.3 配置证书](#33-%E9%85%8D%E7%BD%AE%E8%AF%81%E4%B9%A6)
    - [4. 向CARSI联盟提交IdP配置信息](#4-%E5%90%91carsi%E8%81%94%E7%9B%9F%E6%8F%90%E4%BA%A4idp%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF)
    - [5. 接入本地认证系统](#5-%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F)
    - [6. 释放用户属性](#6-%E9%87%8A%E6%94%BE%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7)
    - [7. 用户登录页面添加/取消隐私保护功能](#7-%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2%E6%B7%BB%E5%8A%A0%E5%8F%96%E6%B6%88%E9%9A%90%E7%A7%81%E4%BF%9D%E6%8A%A4%E5%8A%9F%E8%83%BD)
    - [8. IdP认证测试](#8-idp%E8%AE%A4%E8%AF%81%E6%B5%8B%E8%AF%95)
    - [9. IdP服务web界面自定义](#9-idp%E6%9C%8D%E5%8A%A1web%E7%95%8C%E9%9D%A2%E8%87%AA%E5%AE%9A%E4%B9%89)
    - [10. 日志功能&分析配置](#10-%E6%97%A5%E5%BF%97%E5%8A%9F%E8%83%BD%E5%88%86%E6%9E%90%E9%85%8D%E7%BD%AE)
    - [11. IdP上线运行](#11-idp%E4%B8%8A%E7%BA%BF%E8%BF%90%E8%A1%8C)
  - [常见问题](#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP 安装配置文档 (详细)



本文档为根据Shibboleth原始安装包v3.4.3(可在 http://shibboleth.net/downloads/identity-provider/ 下载)进行安装配置IdP提供指导

## 目标

安装Shibboleth IdP，加入CERNET资源共享服务CARSI，为本校师生访问国内外其他共享资源提供身份认证服务。

## 安装准备

- 系统环境：CentOS 7 64bit 最小安装，内存≥4G，硬盘≥50G；

- IdP域名（网络中心维护，建议：idp.xxx.edu.cn；图书馆维护，建议：idp-lib.xxx.edu.cn），以及对应的https证书，域名一经确定，安装配置后无法修改；

- 网络可通达校园网外网，如果机器前面有防火墙，需要开通机器的TCP 80，443，8443端口；

- 时间同步服务器，以ntp.aliyun.com为例，可根据校园网网络情况进行调整；

- 下载相关[附件](https://mgmt.carsi.edu.cn/frontend/web/docs/carsi-idp-installation-manual.zip)（IdP安装包及本文档提及的相关文件）。

## 安装步骤

### 1. 安装前准备

#### 1.1 修改本机hostname

```
[root@www ~]# vi /etc/hostname
#将localhost.localdomain改成主机域名，例如idp.xxx.xxx.xxx
#重启
[root@www ~]# reboot
```

#### 1.2 关闭selinux

```
#关闭SELinux
[root@www ~]#setenforce 0

#关闭开机启动SELinux
[root@www ~]# vi /etc/selinux/config
# line 7：修改为
SELINUX=disable

# 查看当前selinux状态
[root@www ~]# getenforce
Permissive #表示selinux已关闭
```

#### 1.3 开放本机端口

IdP本机开放80、443和8443端口，443端口提供web服务，8443用于与其他SP（应用资源提供者）后台通信。80端口用于某些HTTPS证书在更新时的连接性测试（如Let's Encrypt），可根据实际情况选择是否开启。

本机防火墙上开放相应端口（http和https分别对应80和443）

```
[root@www ~]# firewall-cmd --add-service=http --permanent
[root@www ~]# firewall-cmd --add-service=https --permanent
[root@www ~]# firewall-cmd --add-port=8443/tcp --permanent
```

刷新本机防火墙

```
[root@www ~]# firewall-cmd --reload
```

如果本机前面配置有其他防火墙，请联系防火墙管理员开通：外部服务器可访问本机80 、443和8443端口（TCP端口）。

#### 1.4 设置时间同步

Shibboleth IdP需要和其他CARSI组件严格校准时间，以保证服务正常运行。

```
[root@www ~]# yum -y install ntp
[root@www ~]# ntpdate -u ntp.aliyun.com
[root@www ~]# timedatectl set-timezone Asia/Shanghai
```

编辑 /etc/ntp.conf，注释掉默认境外ntp服务器，添加国内时间同步服务器（如阿里云）

```
[root@www ~]# vi /etc/ntp.conf
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server ntp.aliyun.com
```

让时间同步开机启动，并且查看当前状态

```
[root@www ~]# chkconfig ntpd on
[root@www ~]# service ntpd start
[root@www ~]# ntpdc -p
```

### 2. 安装IdP运行环境

安装apache、ssl、java、tomcat等IdP运行环境。

安装apache、mod_ssl、java8、tomcat、wget：

```
[root@www ~]# yum -y install httpd
[root@www ~]# rm -f /etc/httpd/conf.d/welcome.conf #删除apache欢迎页面
[root@www ~]# rm -f /etc/httpd/conf.d/autoindex.conf #避免apache目录遍历漏洞
[root@www ~]# vi /etc/httpd/conf/httpd.conf
# line 86: 修改成管理员邮箱
 ServerAdmin xxx@xxx.xxx.xxx //运维联系人邮箱

# line 95: 修改成对应的域名
 ServerName idp.xxx.xxx.xxx:80

# line 151: 改成
 AllowOverride All

# line 164: 增加默认页面扩展名
 DirectoryIndex index.html index.cgi index.php

# 在配置文件最后加上
 ServerTokens Prod
 KeepAlive On

[root@www ~]# yum -y install mod_ssl java-1.8.0-openjdk java-1.8.0-openjdk-devel
[root@www ~]# yum -y install tomcat wget
```

配置java环境变量：

```
[root@www ~]# vi /etc/profile
# 在文件最后加上
export JAVA_HOME=/etc/alternatives/java_sdk_1.8.0
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

# 刷新全局变量
[root@www ~]# source /etc/profile
```

### 3. 安装&本地配置IdP

#### 3.1 安装IdP

建议使用CARSI联盟网站提供的IdP 3.4.3版本安装包，或从Shibboleth官网下载IdP 3.4.3版本安装包。本文档涉及到的其它文件全部打包在附件中。

请提前确认好Hostname（idp所使用的域名）和Attribute Scope（xxx.edu.cn）参数，安装过程中会根据这两个参数生成多个文件，且安装后无法更改。

```
[root@www ~]# mkdir /root/inst
[root@www ~]# cd /root/inst
[root@www ~]# wget https://shibboleth.net/downloads/identity-provider/latest/shibboleth-identity-provider-${IdP_VERSION}.tar.gz
[root@www ~]# tar xzf shibboleth-identity-provider-${IdP_VERSION}.tar.gz
[root@www ~]# cd /root/inst/shibboleth-identity-provider-${IdP_VERSION}
[root@www ~]# ./bin/install.sh

Source (Distribution) Directory (press <enter> to accept default): [/root/inst/shibboleth-identity-provider-3.x.x] #默认回车

Installation Directory: [/opt/shibboleth-idp] #默认回车

Hostname: [idp.xxx.edu.cn] enter #确认是修改后的域名，无误后回车

SAML EntityID: [https://域名/idp/shibboleth] #默认回车

Attribute Scope: [xxx.edu.cn] #输入学校域名，如xxx.edu.cn 回车

Backchannel PKCS12 Password: #创建后台证书密码

Re-enter password: #再输入一遍

Cookie Encryption Key Password: #创建Cookie加密密码

Re-enter password: #再输入一遍

Warning: /opt/shibboleth-idp/bin does not exist.

Warning: /opt/shibboleth-idp/dist does not exist.

Warning: /opt/shibboleth-idp/doc does not exist.

Warning: /opt/shibboleth-idp/system does not exist.

Warning: /opt/shibboleth-idp/webapp does not exist.

Generating Signing Key, CN = 域名 URI = https://域名/idp/shibboleth ...

...done

Creating Encryption Key, CN = 域名 = https://域名/idp/shibboleth ...

...done

Creating Backchannel keystore, CN = 域名 URI = https://域名/idp/shibboleth ...

...done

Creating cookie encryption key files...

...done

Rebuilding /opt/shibboleth-idp/war/idp.war ...

...done

BUILD SUCCESSFUL

#安装成功
```



#### 3.2 配置apache和tomcat 

将系统生成的为java使用的密钥转换成pem格式，供apache使用

```
[root@www ~]# openssl pkcs12 -in /opt/shibboleth-idp/credentials/idp-backchannel.p12 -out /opt/shibboleth-idp/credentials/idp-backchannel.key -nocerts -nodes

Enter Import Password: #输入安装过程中创建的后台证书密码
```

tomcat配置

```
#新建idp.xml
[root@www ~]# vi /etc/tomcat/Catalina/localhost/idp.xml
<Context docBase="/opt/shibboleth-idp/war/idp.war"
                privileged="true"
                antiResourceLocking="false"
                antiJARLocking="false"
                unpackWAR="false"
                swallowOutput="true" />

#定义新的AJP connector
[root@www ~]# vi /etc/tomcat/server.xml
#line 76，默认已注释掉Connector配置（此处无需修改）
<!--
     <Connector executor="tomcatThreadPool"
                port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
-->

#在此处新增
<Connector port="8009" address="127.0.0.1"
               enableLookups="false" redirectPort="443" protocol="AJP/1.3"
               tomcatAuthentication="false" />
```

因为tomcat7不支持JSTL，需要附件的javax.servlet.jsp.jstl-api-1.2.1.jar和javax.servlet.jsp.jstl-1.2.1.jar，放入tomcat的/usr/share/tomcat/lib/路径下。 

apache配置

```
#新建
[root@www ~]# vi /var/www/html/index.html
<script language="javascript"
type="text/javascript">
   window.location.href="/idp";
</script>

#新建
[root@www ~]# vi /etc/httpd/conf.d/ports.conf
Listen 443
Listen 8443

#修改
[root@www ~]# vi /etc/httpd/conf/httpd.conf
#Line 42，注释掉
#Listen 80

#备份ssl.conf
[root@www ~]# cp /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.dist
```

将附件idp.conf和idp8443.conf，放入/etc/httpd/conf.d文件夹中

```
[root@www ~]# vi /etc/httpd/conf.d/ssl.conf
#Line 5，注释掉
# Listen 443 https
#删掉整个<VirtualHost>部分，line56-line216
```

重启tomcat和apache

```
[root@www ~]# chown -R tomcat.tomcat /opt/shibboleth-idp
#启动tomcat和apache
[root@www ~]# systemctl start tomcat
[root@www ~]# systemctl enable tomcat
[root@www ~]# systemctl start httpd 
[root@www ~]# systemctl enable httpd
```

访问https://IP地址或者域名/idp 如果看到页面内容为

![CARSI](/CARSI_IdP安装配置文档_详细.files/001.png)

则表示IdP安装成功。注：此时网站使用的是自签的证书，因此使用浏览器访问可能会提示站点不安全，这时选择继续访问即可。

#### 3.3 配置证书

采用学校提供的证书，并配置证书路径

```
[root@www ~]# vi /etc/httpd/conf.d/idp.conf
# line 17、18: 改为
SSLCertificateFile cert证书绝对路径cert.pem
SSLCertificateKeyFile privkey绝对路径privkey.pem

[root@www ~]# systemctl restart httpd
[root@www ~]# systemctl restart tomcat
```

### 4. 向CARSI联盟提交IdP配置信息

将/opt/shibboleth-idp/metadata/idp-metadata.xml文件下载到本地。

登陆 [CARSI会员自服务系统](https://mgmt.carsi.edu.cn) 用户名为申请时填的学校域名，密码为申请时填的项目负责人的手机号。

![CARSI](/CARSI_IdP安装配置文档_详细.files/002.png)         

在“我的CARSI->我的IdP”中，选择“上传Metadata”完成该文件的上传，上传成功后该页面会显示“已提供”。

![CARSI](/CARSI_IdP安装配置文档_详细.files/003.png)

在预上线环境配置。下载https://dspre.carsi.edu.cn/carsifed-metadata-pre.xml 文件，放入/opt/shibboleth-idp/metadata文件夹，并且修改文件的所属用户和组。

```
[root@www ~]# chown -R tomcat.tomcat /opt/shibboleth-idp
```

将联盟提供的metadata验证证书dsmeta.pem放入/opt/shibboleth-idp/credentials目录下。修改metadata-providers.xml

```
[root@www ~]# vi /opt/shibboleth-idp/conf/metadata-providers.xml

#在</MetadataProvider>内新增
<MetadataProvider id="HTTPMetadata"
          xsi:type="FileBackedHTTPMetadataProvider"
          backingFile="/opt/shibboleth-idp/metadata/carsifed-metadata-pre.xml"
          metadataURL="https://dspre.carsi.edu.cn/carsifed-metadata-pre.xml"> 

         <MetadataFilter xsi:type="SignatureValidation" certificateFile="/opt/shibboleth-idp/credentials/dsmeta.pem" />
         <MetadataFilter xsi:type="EntityRoleWhiteList">
             <RetainedRole>md:SPSSODescriptor</RetainedRole>
         </MetadataFilter>
     </MetadataProvider>
```



### 5. 接入本地认证系统

请参考文档[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“接入本地认证系统”进行配置，从LDAP、OAuth、CAS中根据学校情况选择一种进行接入，默认使用LDAP。

### 6. 释放用户属性

请参考文档[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“释放用户属性”进行配置。

### 7. 用户登录页面添加/取消隐私保护功能

请参考文档[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“用户登录页面添加/取消隐私保护功能”进行配置。

### 8. IdP认证测试

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，在预上线环境进行测试，尤其注意属性释放是否成功。

### 9. IdP服务web界面自定义

请参考文档[CARSI_IdP界面简单定制.md](CARSI_IdP界面简单定制.md) 了解简单定制图标、文本的方法。

所有界面的模板都在/opt/shibboleth-idp/views路径下，可以根据需要自行深度定制，包括本校用户IdP登陆界面、隐私保护界面、错误提示界面等，可根据本校情况自行修改。

### 10. 日志功能&分析配置

在Shibboleth IdP配置日志统计功能之后，可通过[CARSI会员自服务系统](https://mgmt.carsi.edu.cn)  查看本IdP运行和被访问情况。如果不进行相关配置，则在[CARSI会员自服务系统](https://mgmt.carsi.edu.cn)  无法看到相关统计内容。

```
[root@www ~]# vi /opt/shibboleth-idp/conf/audit.xml
#line 18 替换
<entry key="Shibboleth-Audit" value="%T|%b|%I|%SP|%P|%IdP|%bb|%III|%u|%ac|%attr|%n|%i|%a|%s|" />

#取消注释 
<bean id="shibboleth.AuditDateTimeFormat" class="java.lang.String" c:_0="YYYY-MM-dd'T'HH:mm:ss.SSSZZ" />
<util:constant id="shibboleth.AuditDefaultTimeZone" static-field="java.lang.Boolean.TRUE" />

[root@www ~]# mkdir /var/www/html/auditlog
[root@www ~]# vi /etc/httpd/conf/httpd.conf
#line 144 修改
Options FollowSymLinks

#line 157 </Directory> 后增加
<Directory "/var/www/html/auditlog">
     Order deny,allow
     Deny from all
     Allow from 115.27.243.6
</Directory>

#新建
[root@www ~]# vi /var/www/html/auditlog/auditlog.sh
rm -rf /var/www/html/auditlog/auditlog-`date -d -2hours +%Y-%m-%d-%H`.log
grep `date -d -1hours +%Y-%m-%dT%H` /opt/shibboleth-idp/logs/idp-audit.log > /var/www/html/auditlog/auditlog-`date -d -1hours +%Y-%m-%d-%H`.log

#添加定时任务
[root@www ~]# crontab -e
0 */1 * * * sh /var/www/html/auditlog/auditlog.sh >/dev/null 2>&1

#重启tomcat和apache
[root@www ~]# systemctl restart tomcat
[root@www ~]# systemctl restart httpd
```



### 11. IdP上线运行

在预上线环境（https://dspre.carsi.edu.cn/) 试验访问成功后，请发送邮件给 carsi@pku.edu.cn，申请在CARSI产品环境上线。待收到上线成功邮件通知后，下载https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml 文件，放入/opt/shibboleth-idp/metadata文件夹，并且修改文件的所属用户和组。

```
[root@www ~]# chown -R tomcat.tomcat /opt/shibboleth-idp
[root@www ~]# vi /opt/shibboleth-idp/conf/metadata-providers.xml
#修改之前定义的MetadataProvider
<MetadataProvider id="HTTPMetadata"
          xsi:type="FileBackedHTTPMetadataProvider"
          backingFile="/opt/shibboleth-idp/metadata/carsifed-metadata.xml"
          metadataURL="https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml"> 

         <MetadataFilter xsi:type="SignatureValidation" certificateFile="/opt/shibboleth-idp/credentials/dsmeta.pem" />
         <MetadataFilter xsi:type="EntityRoleWhiteList">
             <RetainedRole>md:SPSSODescriptor</RetainedRole>
         </MetadataFilter>
</MetadataProvider>

[root@www ~]# systemctl restart tomcat
```

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，在线上环境进行测试，尤其注意属性释放是否成功。

## 常见问题

（1）出现Security Problem（TODO：加截屏），请确认一下IdP的时间是否同步；

（2）如果第一次安装失败，可以把/opt/shibboleth-idp文件夹整个删除，重新按照流程安装一遍；

（3）如果出现其他问题，可以查看shibboleth和tomcat日志，shibboleth日志位置在/opt/shibboleth-idp/logs，tomcat日志在/var/log/tomcat下。





 

 
