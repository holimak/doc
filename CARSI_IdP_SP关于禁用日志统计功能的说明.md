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

