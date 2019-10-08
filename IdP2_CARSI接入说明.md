<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [IdP2 CARSI 接入说明](#idp2-carsi-%E6%8E%A5%E5%85%A5%E8%AF%B4%E6%98%8E)
  - [增加 Metadata 源](#%E5%A2%9E%E5%8A%A0-metadata-%E6%BA%90)
  - [属性释放](#%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE)
    - [增加属性获取](#%E5%A2%9E%E5%8A%A0%E5%B1%9E%E6%80%A7%E8%8E%B7%E5%8F%96)
    - [允许属性释放](#%E5%85%81%E8%AE%B8%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE)
    - [适配 scope 域名](#%E9%80%82%E9%85%8D-scope-%E5%9F%9F%E5%90%8D)
  - [重启 IdP](#%E9%87%8D%E5%90%AF-idp)
  - [提交 Metadata](#%E6%8F%90%E4%BA%A4-metadata)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# IdP2 CARSI 接入说明
本文档介绍如何在 IdP2 版本下修改配置接入 CARSI。注意 Shibboleth 已经停止对 IdP2 的维护和支持，新部署 IdP 的用户请选择直接部署 IdP3 版本。本文档供已有 IdP2 在其他 Shibboleth 联盟内使用，尚未升级 IdP3 之前，接入 carsi 的方案。

我们仍然建议尽快升级您的 IdP 到 IdP3。

## 增加 Metadata 源
Shibboleth 联盟本质上是一个点对点的互相信任关系。IdP 允许配置多个 metadataprovider，所以 IdP 只需要引入 对应联盟的 metadata ，就可以同时支持在多个联盟内进行联盟认证。修改 `relying-party.xml` 配置。增加额外的 metadata 获取配置。
```
         <metadata:MetadataProvider id="CARSIMD" xsi:type="metadata:FileBackedHTTPMetadataProvider"
                          metadataURL="https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml"
                          backingFile="/opt/idp/metadata/carsi.xml"/>
```

由于 IdP2 部署较早，如果操作系统的 openssl 版本较低，可能会有 SSL 握手失败的问题，我们可以人工的去下载 metadata，然后以 Filesystem 的方式加载
```
2019-07-10 01:08:29,537 - ERROR [org.opensaml.saml2.metadata.provider.HTTPMetadataProvider:273] - Error retrieving metadata from https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml|
javax.net.ssl.SSLException: SSLSession was invalid: Likely implicit handshake failure: Set system property javax.net.debug=all for details
```
```
         <metadata:MetadataProvider id="CARSIMD" xsi:type="metadata:FilesystemMetadataProvider"
                          metadataFile="/opt/idp/metadata/carsi.xml"
                          maxRefreshDelay="P1D" />
```
同时，做一个系统的定时任务去定时更新 metadata 就好了
```
15 5 * * * wget --no-check-certificate https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml -O /opt/idp/metadata/carsi.xml
```

## 属性释放
### 增加属性获取
修改 `attribute-resolver.xml` 配置，增加 carsi 所需的属性获取。此处与 IdP3 基本一致。
```
    <resolver:AttributeDefinition xsi:type="ad:Scoped" id="eduPersonPrincipalName" scope="example.edu.cn" sourceAttributeID="usertype">
        <resolver:Dependency ref="myldap" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eduPersonPrincipalName" />
    </resolver:AttributeDefinition>
    <resolver:AttributeDefinition id="eduPersonScopedAffiliation" xsi:type="ad:Scoped" scope="example.edu.cn" sourceAttributeID="eduPersonScopedAffiliation">
        <resolver:Dependency ref="myldap" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" />
    </resolver:AttributeDefinition>
```

### 允许属性释放
修改 `attribute-filter.xml`，增加新增属性的过滤配置，允许释放。
```
        <afp:AttributeRule attributeID="eduPersonScopedAffiliation">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
        <afp:AttributeRule attributeID="eduPersonPrincipalName">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
```
### 适配 scope 域名
修改 `idp-metadata.xml`，修改 scope 部分的域名与您学校的域名适配。否则带 scope 的属性会被 sp 过滤掉。
```
        <Extensions>
            <shibmd:Scope regexp="false">example.edu.cn</shibmd:Scope>
        </Extensions>
```

## 重启 IdP
根据 IdP 的部署模式，重启您的 java 容器，通常是 tomcat。确认 IdP 日志正常没有报错。

## 提交 Metadata
将 IdP 的 metadata 提交给 carsi，并在 carsi 的 sp 上测试。如果一切顺利，恭喜你已经完成了 carsi 的接入。

记得要筹划 IdP3 的升级哦。