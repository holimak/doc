<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [IdP环境验证](#idp%E7%8E%AF%E5%A2%83%E9%AA%8C%E8%AF%81)
  - [预上线环境](#%E9%A2%84%E4%B8%8A%E7%BA%BF%E7%8E%AF%E5%A2%83)
  - [线上环境](#%E7%BA%BF%E4%B8%8A%E7%8E%AF%E5%A2%83)
  - [常见问题](#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP认证测试



## 预上线环境

在浏览器中打开一个新的无痕窗口。访问：https://sptest.pku.edu.cn/secure 浏览器跳转到学校选择页面，选择要测试的学校。

![CARSI](/CARSI_IdP环境验证.files/001.png)

跳转到学校的登录页面，填入相关信息。如北京大学测试IdP的登录页面：

![CARSI](/CARSI_IdP环境验证.files/002.png)

根据IdP的配置，可能会出现条款页面（该页面也可配置隐藏），这里需要同意相关条款。

![CARSI](/CARSI_IdP环境验证.files/003.png)

根据IdP的配置，可能会出现属性释放页面（该页面也可配置隐藏），在这里用户可以选择性地去掉对某些属性的释放。

![CARSI](/CARSI_IdP环境验证.files/004.png)

登录成功后，浏览器跳转回https://sptest.pku.edu.cn/secure/ 页面，该页面最后一行如提示“恭喜，您已认证成功！当前页面为SPTEST模拟的资源页面！”，则代表登陆成功。

![CARSI](/CARSI_IdP环境验证.files/005.png)

最后验证一下属性释放是否成功：访问https://sptest.pku.edu.cn/Shibboleth.sso/Session 应返回类似如下的信息，最后两行应显示SP获得到的属性及取值。

```
Miscellaneous
Session Expiration (barring inactivity): 479 minute(s)
Client Address: 162.105.126.239
SSO Protocol: urn:oasis:names:tc:SAML:2.0:protocol
Identity Provider: https://idp3.pku.edu.cn/idp/shibboleth
Authentication Time: 2019-09-20T09:37:36.708Z
Authentication Context Class: urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
Authentication Context Decl: (none)

Attributes
affiliation: faculty@pku.edu.cn
eppn: carsitest@pku.edu.cn
```

## 线上环境

在浏览器中打开一个新的无痕窗口。访问：https://ds.carsi.edu.cn/

浏览器跳转到学校选择页面，选择要测试的学校，跳转到学校的登录页面，填入相关信息进行验证。IdP相关页面可参考上文“预上线环境”。

登录成功后，浏览器跳转回https://ds.carsi.edu.cn/secure/ 页面，该页面如提示“恭喜，您已认证成功！”，则代表登陆成功。

最后验证一下属性释放是否成功：访问https://ds.carsi.edu.cn/Shibboleth.sso/Session 应返回类似如下的信息，最后两行应显示SP获得到的属性，但线上环境出于安全考虑并未配置明文获取，因此看不到具体的取值。

```
Miscellaneous
Session Expiration (barring inactivity): 479 minute(s)
Client Address: 2001:da8:201:1126:9dd3:6a5d:4a29:be24
SSO Protocol: urn:oasis:names:tc:SAML:2.0:protocol
Identity Provider: https://idp.pku.edu.cn/idp/shibboleth
Authentication Time: 2019-09-20T09:41:32.563Z
Authentication Context Class: urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
Authentication Context Decl: (none)

Attributes
affiliation: 1 value(s)
eppn: 1 value(s)
```

## 常见问题

[CARSI_IdP属性释放常见问题.md](CARSI_IdP属性释放常见问题.md) 