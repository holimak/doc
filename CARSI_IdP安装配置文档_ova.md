<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CARSI IdP 安装配置文档 (ova)](#carsi-idp-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E6%96%87%E6%A1%A3-ova)
  - [目标](#%E7%9B%AE%E6%A0%87)
  - [安装准备](#%E5%AE%89%E8%A3%85%E5%87%86%E5%A4%87)
  - [安装步骤](#%E5%AE%89%E8%A3%85%E6%AD%A5%E9%AA%A4)
    - [1. 安装前准备](#1-%E5%AE%89%E8%A3%85%E5%89%8D%E5%87%86%E5%A4%87)
      - [1.1 设置时间同步](#11-%E8%AE%BE%E7%BD%AE%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5)
    - [2. 安装&本地配置IdP](#2-%E5%AE%89%E8%A3%85%E6%9C%AC%E5%9C%B0%E9%85%8D%E7%BD%AEidp)
      - [2.1 安装IdP](#21-%E5%AE%89%E8%A3%85idp)
      - [2.2 配置证书](#22-%E9%85%8D%E7%BD%AE%E8%AF%81%E4%B9%A6)
      - [2.3 向CARSI联盟提交IdP配置信息](#23-%E5%90%91carsi%E8%81%94%E7%9B%9F%E6%8F%90%E4%BA%A4idp%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF)
    - [3. 接入本地认证系统](#3-%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F)
      - [3.1 接入本地认证系统](#31-%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F)
      - [3.2 用户登录页面添加/取消隐私保护功能](#32-%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2%E6%B7%BB%E5%8A%A0%E5%8F%96%E6%B6%88%E9%9A%90%E7%A7%81%E4%BF%9D%E6%8A%A4%E5%8A%9F%E8%83%BD)
    - [4. IdP认证测试](#4-idp%E8%AE%A4%E8%AF%81%E6%B5%8B%E8%AF%95)
    - [5. IdP服务web界面自定义](#5-idp%E6%9C%8D%E5%8A%A1web%E7%95%8C%E9%9D%A2%E8%87%AA%E5%AE%9A%E4%B9%89)
  - [IdP上线运行](#idp%E4%B8%8A%E7%BA%BF%E8%BF%90%E8%A1%8C)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

 

# CARSI IdP 安装配置文档 (ova)



本文档为根据北京大学CARSI运行小组提供的ova虚机镜像进行安装配置IdP提供指导

## 目标

安装Shibboleth IdP，加入CERNET资源共享服务CARSI，为本校师生访问国内外其他共享资源提供身份认证服务。

## 安装准备

- IdP域名（网络中心维护，建议：idp.xxx.edu.cn；图书馆维护，建议：idp-lib.xxx.edu.cn），以及对应的https证书，域名一经确定，安装配置后无法修改；
- 网络可通达校园网外网，如果机器前面有防火墙，需要开通机器的TCP 80，443，8443端口；
- 下载[idp3_ldap_install_v1.0.ova](https://mgmt.carsi.edu.cn/frontend/web/docs/idp3_ldap_install_v1.0.ova)文件，导入exsi虚拟机中（虚机为linux环境，其root密码请发邮件至carsi@pku.edu.cn 获取，获取后请尽快修改密码）。
- 下载相关[附件](https://mgmt.carsi.edu.cn/frontend/web/docs/carsi-idp-installation-manual.zip)（IdP安装包及本文档提及的相关文件）。

## 安装步骤

### 1. 安装前准备

```
#修改默认密码
[root@www ~]# passwd
#输入两次新密码

#配置网络
[root@www ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens160
IPADDR=IP地址
NETMASK=子网掩码
GATEWAY=默认网关
DNS1=DNS服务器
[root@www ~]# systemctl restart network

#修改主机名
[root@www ~]# vi /etc/hostname
#将localhost.localdomain改成主机域名，例如idp.xxx.xxx.xxx，然后重启
[root@www ~]# reboot
```



#### 1.1 设置时间同步

Shibboleth IdP需要和其他CARSI组件严格校准时间，以保证服务正常运行。现在时间同步服务器默认设置为server ntp.aliyun.com，如需要更改：

编辑 /etc/ntp.conf，将ntp.aliyun.com更改为新的时间同步服务器，并重启。

```
[root@www ~]# vi /etc/ntp.conf
server ntp.aliyun.com
[root@www ~]# service ntpd restart
```



### 2. 安装&本地配置IdP

#### 2.1 安装IdP
请提前确认好Hostname（IdP所使用的域名）和Attribute Scope（xxx.edu.cn）参数，安装过程中会根据这两个参数生成多个文件，且安装后无法更改。

```
[root@www ~]# sh /root/inst/idp3config/autoconfig.sh

Source (Distribution) Directory (press <enter> to accept default): [/root/inst/shibboleth-identity-provider-3.x.x] #默认回车

Installation Directory: [/opt/shibboleth-idp] #默认回车

Hostname: [idp.xxx.edu.cn] enter #确认是修改后的域名，无误后回车

SAML EntityID: [https://idp.xxx.edu.cn/idp/shibboleth] #默认回车

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

Enter Import Password: #输入安装过程中创建的后台证书密码

#安装成功
```



#### 2.2 配置证书

采用学校提供的证书，并配置证书路径

```
[root@www ~]# vi /etc/httpd/conf.d/idp.conf
# line 17、18: 改为
SSLCertificateFile cert证书绝对路径cert.pem
SSLCertificateKeyFile privkey绝对路径privkey.pem

#完成后重启服务
[root@www ~]# systemctl restart httpd
[root@www ~]# systemctl restart tomcat
```



#### 2.3 向CARSI联盟提交IdP配置信息

将/opt/shibboleth-idp/metadata/idp-metadata.xml文件下载到本地。

登陆 [CARSI会员自服务系统](https://mgmt.carsi.edu.cn) 用户名为申请时填的学校域名，密码为申请时填的项目负责人的手机号。

![CARSI](/CARSI_IdP安装配置文档_ova.files/001.png)

在“我的CARSI->我的IdP”中，选择“上传Metadata”完成该文件的上传，上传成功后该页面会显示“已提供”。

![CARSI](/CARSI_IdP安装配置文档_ova.files/002.png)

### 3. 接入本地认证系统

#### 3.1 接入本地认证系统

请参考文档[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“接入本地认证系统”进行配置，从LDAP、OAuth、CAS中根据学校情况选择一种进行接入，默认使用LDAP。

注：[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“释放用户属性”部分，默认在虚机镜像中已配置完成，因此无需配置。

#### 3.2 用户登录页面添加/取消隐私保护功能

默认在虚机镜像中已配置完成，无需配置。

如果需要修改，请参考文档[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) 中的“用户登录页面添加/取消隐私保护功能”进行配置。

### 4. IdP认证测试

```
#启动IdP
[root@www ~]# sh /root/inst/idp3config/startidp.sh
```

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，在预上线环境进行测试，尤其注意属性释放是否成功。

### 5. IdP服务web界面自定义

请参考文档[CARSI_IdP界面简单定制.md](CARSI_IdP界面简单定制.md) 了解简单定制图标、文本的方法。

所有界面的模板都在/opt/shibboleth-idp/views路径下，可以根据需要自行深度定制，包括本校用户IdP登陆界面、隐私保护界面、错误提示界面等，可根据本校情况自行修改。

## IdP上线运行

在预上线环境（https://dspre.carsi.edu.cn/) 试验访问成功后，请发送邮件给 carsi@pku.edu.cn，申请在CARSI线上环境上线，邮件中请将SP属性释放成功的截图作为附件提供（可参考下图）。

![CARSI](/CARSI_IdP安装配置文档_ova.files/verified.png)

待收到上线成功邮件通知后，执行以下脚本完成上线。

```
[root@www ~]# sh /root/inst/idp3config/onlineidp.sh
```

请参考文档[CARSI_IdP认证测试.md](CARSI_IdP认证测试.md)，在线上环境进行测试，尤其注意属性释放是否成功。

