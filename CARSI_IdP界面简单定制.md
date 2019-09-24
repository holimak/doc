<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Shibboleth IdP界面定制](#shibboleth-idp%E7%95%8C%E9%9D%A2%E5%AE%9A%E5%88%B6)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP界面定制

感谢山东大学秦丰林老师提供！

定制登录界面Logo和文字：

```
vi /opt/shibboleth-idp/messages/messages.properties

idp.title = 教育网统一认证与资源共享（Carsi）
idp.title.suffix = 错误
idp.footer = 教育网统一认证与资源共享-山东大学
idp.logo = /images/sdulogo.jpg
idp.login.loginTo = 登录到
idp.login.username = 账号
idp.login.password = 密码
idp.login.donotcache = 不保存账号信息
idp.login.login = 登录
idp.login.pleasewait = 正在登录，请等待...
idp.attribute-release.revoke = 清除历史授权信息
```

idp.logo中的logo图像文件在/opt/shibboleth-idp/edit-webapp/images/目录里
修改图像文件后需要重新编译：

```
/opt/shibboleth-idp/bin/build.sh
```

然后重启tomcat：

```
systemctl restart tomcat
```

然后重启tomcat
systemctl restart tomcat

