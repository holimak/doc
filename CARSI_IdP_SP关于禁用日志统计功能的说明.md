<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CARSI IDP/SP关于禁用日志统计功能的说明](#carsi-idpsp%E5%85%B3%E4%BA%8E%E7%A6%81%E7%94%A8%E6%97%A5%E5%BF%97%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD%E7%9A%84%E8%AF%B4%E6%98%8E)
  - [背景说明](#%E8%83%8C%E6%99%AF%E8%AF%B4%E6%98%8E)
  - [关闭日志上报功能](#%E5%85%B3%E9%97%AD%E6%97%A5%E5%BF%97%E4%B8%8A%E6%8A%A5%E5%8A%9F%E8%83%BD)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IDP/SP关于禁用日志统计功能的说明

## 背景说明

使用《[CARSI_IdP安装配置文档_ova.md](CARSI_IdP安装配置文档_ova.md) 》或《[CARSI_IdP安装配置文档_详细.md](CARSI_IdP安装配置文档_详细.md) 》，《[CARSI_SP安装配置文档.md](CARSI_SP安装配置文档.md) 》安装配置的IdP或SP服务，默认配置了日志上报功能，用户可登录[CARSI会员自服务系统](https://mgmt.carsi.edu.cn)  后查看基于上报日志的统计信息。如需关闭日志上报功能，可执行以下步骤。

 

## 关闭日志上报功能

IdP：

```
[root@www ~]# crontab -e
#删除这一行
0 */1 * * * sh /var/www/html/auditlog/auditlog.sh >/dev/null 2>&1
```

SP：

```
[root@www ~]# crontab -e
#删除这一行
0 */1 * * * sh /var/www/html/transactionlog/transactionlog.sh >/dev/null 2>&1
```

