# CARSI SP 安装配置文档

本文档为根据Shibboleth原始安装包（通过网络安装，不需要提前下载安装包）进行安装配置SP提供指导

## 目标

安装Shibboleth SP，加入CERNET资源共享服务CARSI，共享本校资源，以供其他CARSI高校用户或者国外用户访问。

## 安装准备

- 系统环境：Centos7 64bit 最小安装，内存≥4G，硬盘≥50G；
- SP域名：建议sp-xxx.xxx.edu.cn，以及对应的https证书；
- 网络可通达校园网外网，如果机器前面有防火墙，需要开通机器的TCP 80，443端口；
- 时间同步服务器，以ntp.aliyun.com为例，可根据校园网网络情况进行调整；
- 通过网络安装，不需要提前下载安装包。

## 安装步骤

### 1. 安装前准备

#### 1.1 修改本机hostname

```
[root@www ~]# vi /etc/hostname
#将localhost.localdomain改成主机域名，例如
sp-xxx.xxx.edu.cn
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



#### 1.3端口开放

SP本机开放80和443端口，可访问外部服务器8443端口。443端口提供web服务，访问外部8443用于与其他IdP后台通信。80端口用于某些HTTPS证书在更新时的连接性测试（如Let's Encrypt），可根据实际情况选择是否开启。

本机防火墙上开放相应端口（http和https分别对应80和443）

```
[root@www ~]# firewall-cmd --add-service=http --permanent
[root@www ~]# firewall-cmd --add-service=https --permanent
```

刷新本机防火墙

```
[root@www ~]# firewall-cmd --reload
```

如果本机前面配置有其他防火墙，请联系防火墙管理员开通

1）外部服务器可访问本机80和443端口（TCP端口）；

2）本机可访问外部系统8443端口（TCP端口），用于SP和IdP后台通信。

#### 1.4 设置时间同步

CARSI SP需要和其他CARSI组件严格校准时间，以保证服务正常运行。

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
[root@www ~]#chkconfig ntpd on
[root@www ~]#service ntpd start
[root@www ~]#ntpdc -p
```

### 2. 运行环境安装

#### 2.1 apache安装

apache是CARSI SP基础运行环境。

```
[root@www ~]# yum -y install httpd
[root@www ~]# rm -f /etc/httpd/conf.d/welcome.conf #删除apache欢迎页面
[root@www ~]# rm -f /etc/httpd/conf.d/autoindex.conf #避免apache目录遍历漏洞
[root@www ~]# vi /etc/httpd/conf/httpd.conf

# line 86: 修改成管理员邮箱
 ServerAdmin xxx@xxx.edu.cn

# line 95: 取消注释，并修改成对应的域名
 ServerName xxx.xxx.edu.cn:80

# line 151: <Directory "/var/www/html">对应的改成
 AllowOverride All

# line 164: 增加默认页面扩展名
 DirectoryIndex index.html index.cgi index.php

# 在配置文件最后加上
 ServerTokens Prod
 KeepAlive On

[root@www ~]# systemctl start httpd
[root@www ~]# systemctl enable httpd
```



#### 2.2 SSL安装

用https的方式来访问CARSI SP服务，提前准备好https证书。

```
[root@www ~]# yum -y install mod_ssl
[root@www ~]# vi /etc/httpd/conf.d/ssl.conf
# line 59: 取消注释
DocumentRoot "/var/www/html"

# line 60: 取消注释并且加上对应的域名
ServerName xxx.xxx.edu.cn:443

# line 75: 改成
SSLProtocol -All +TLSv1 +TLSv1.1 +TLSv1.2

# line 100: 改成对应证书crt路径
SSLCertificateFile cert.pem证书绝对路径

# line 107: 改成对应证书key路径
SSLCertificateKeyFile privkey.pem绝对路径

# （如有Chain文件）line 116: 改成对应证书Chain路径
SSLCertificateChainFile chain.pem绝对路径

[root@www ~]# systemctl restart httpd

#配置http向https跳转
[root@www ~]# vi /etc/httpd/conf.d/vhost.conf
<VirtualHost *:80>
     DocumentRoot /var/www/html
     ServerName xxx.xxx.edu.cn
     RewriteEngine On
     RewriteCond %{HTTPS} off
     RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
</VirtualHost>

[root@www ~]# systemctl restart httpd
```



### 3. SP安装&本地配置

通过网络方式安装CARSI SP软件，按照CARSI标准进行配置。

```
#通过yum源的方式安装
[root@www ~]# wget http://download.opensuse.org/repositories/security://shibboleth/CentOS_7/security:shibboleth.repo -P /etc/yum.repos.d
[root@www ~]# yum install shibboleth
[root@www ~]# systemctl start shibd  
[root@www ~]# systemctl enable shibd
[root@www ~]# systemctl restart httpd

#配置SP受保护资源目录
[root@www ~]# vi /etc/httpd/conf.d/shib.conf
#line 49
<Location /secure> /secure 指的是受保护资源的目录，按照需要自行修改

#配置SP entityID
[root@www ~]# vi /etc/shibboleth/shibboleth2.xml

#将：
ApplicationDefaults entityID="https://sp.example.org/shibboleth"
#改为：
ApplicationDefaults entityID="https://[sp域名]/shibboleth"

#将
<SSO entityID="https://idp.example.org/idp/shibboleth" discoveryProtocol="SAMLDS" discoveryURL="https://ds.example.org/DS/WAYF">
               SAML2
</SSO>
#改为
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://dspre.carsi.edu.cn/ds/index.html">
               SAML2
</SSO>

#在<ApplicationDefaults>代码块内增加（/etc/shibboleth/carsifed-metadata-pre.xml为待生成的metadata备份文件）

<MetadataProvider type="XML" url="https://dspre.carsi.edu.cn/carsifed-metadata-pre.xml"            backingFilePath="/etc/shibboleth/carsifed-metadata-pre.xml" legacyOrgNames="true" reloadInterval="600" >
</MetadataProvider>

[root@www ~]# systemctl start shibd  
[root@www ~]# systemctl enable shibd
[root@www ~]# systemctl restart httpd
```



### 4. 向CARSI联盟提交SP配置信息

SP安装配置完成后，metadata 文件可通过访问URL获取：https://[SP域名]/Shibboleth.sso/Metadata，将文件下载。

登陆 [CARSI会员自服务系统](https://mgmt.carsi.edu.cn)  用户名为申请时填的学校域名，密码为申请时填的项目负责人的手机号。

 ![CARSI](/CARSI_SP安装配置文档.files/001.png)

在“我的CARSI->SP管理”中，选择“添加SP”，按照提示添加SP 并上传metadata文件。

![CARSI](/CARSI_SP安装配置文档.files/002.png)

在预上线环境配置。下载https://dspre.carsi.edu.cn/carsifed-metadata-pre.xml文件，放入/etc/shibboleth/文件夹下。

### 5. 认证测试

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，使用联盟提供的测试IdP及其账号，在预上线环境进行测试，尤其注意属性释放是否成功。

### 6. 日志功能&分析配置

在CARSI SP配置日志统计功能之后，可通过[CARSI会员自服务系统](https://mgmt.carsi.edu.cn)  查看本SP运行和被访问情况。

```
[root@www ~]# vi /etc/shibboleth/shibboleth2.xml
#line 5 替换
<OutOfProcess tranLogFormat="%u|%s|%IDP|%i|%ac|%t|%attr|%n|%b|%E|%S|%SS|%L|%UA|%a|%SP" /> 

[root@www ~]# mkdir /var/www/html/transactionlog
[root@www ~]# vi /etc/httpd/conf/httpd.conf
#line 144 修改
Options FollowSymLinks

#line 157 </Directory> 后增加
<Directory "/var/www/html/transactionlog">
     Order deny,allow
     Deny from all
     Allow from 115.27.243.6
</Directory>

#新建
[root@www ~]# vi /var/www/html/transactionlog/transactionlog.sh

rm -rf /var/www/html/transactionlog/transaction-`date -d -2hours +%Y-%m-%d-%H`.log
grep "`date -d -1hours +%Y-%m-%d` `date -d -1hours +%H`" /var/log/shibboleth/transaction.log > /var/www/html/transactionlog/transaction-`date -d -1hours +%Y-%m-%d-%H`.log

#添加定时任务
[root@www ~]# crontab -e
0 */1 * * * sh /var/www/html/transactionlog/transactionlog.sh >/dev/null 2>&1
```



### 7. SP上线运行

在预上线环境（https://dspre.carsi.edu.cn/)试验访问成功后，请发送邮件给 carsi@pku.edu.cn，申请在CARSI产品环境上线。待收到上线成功邮件通知后，下载https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml文件，放入/etc/shibboleth/文件夹，并且修改文件的所属用户和组。

```
[root@www ~]# vi /etc/shibboleth/shibboleth2.xml
#将：
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://dspre.carsi.edu.cn/ds/index.html">
               SAML2
</SSO>

#改为：
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://ds.carsi.edu.cn/ds/index.html">
               SAML2
</SSO>

#将：
<MetadataProvider type="XML" url="https://dspre.carsi.edu.cn/carsifed-metadata-pre.xml"            backingFilePath="/etc/shibboleth/carsifed-metadata.xml" legacyOrgNames="true" reloadInterval="600" >

#改为：
<MetadataProvider type="XML" url="https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml"            backingFilePath="/etc/shibboleth/carsifed-metadata.xml" legacyOrgNames="true" reloadInterval="600" >
</MetadataProvider>

[root@www ~]# systemctl start shibd  
[root@www ~]# systemctl enable shibd
[root@www ~]# systemctl restart httpd
```

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，在线上环境进行测试，尤其注意属性释放是否成功。

