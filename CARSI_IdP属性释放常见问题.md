<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CARSI_IdP属性释放常见问题](#carsi_idp%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
  - [利用SP的Session调试页面确认属性释放](#%E5%88%A9%E7%94%A8sp%E7%9A%84session%E8%B0%83%E8%AF%95%E9%A1%B5%E9%9D%A2%E7%A1%AE%E8%AE%A4%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE)
  - [原理说明](#%E5%8E%9F%E7%90%86%E8%AF%B4%E6%98%8E)
    - [SP端配置及获取属性的原理](#sp%E7%AB%AF%E9%85%8D%E7%BD%AE%E5%8F%8A%E8%8E%B7%E5%8F%96%E5%B1%9E%E6%80%A7%E7%9A%84%E5%8E%9F%E7%90%86)
    - [IdP端配置及释放属性的原理](#idp%E7%AB%AF%E9%85%8D%E7%BD%AE%E5%8F%8A%E9%87%8A%E6%94%BE%E5%B1%9E%E6%80%A7%E7%9A%84%E5%8E%9F%E7%90%86)
  - [常见问题](#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
    - [问题一（无Session）](#%E9%97%AE%E9%A2%98%E4%B8%80%E6%97%A0session)
    - [问题二（看不到具体的值）](#%E9%97%AE%E9%A2%98%E4%BA%8C%E7%9C%8B%E4%B8%8D%E5%88%B0%E5%85%B7%E4%BD%93%E7%9A%84%E5%80%BC)
    - [问题三（SP获取不到属性）](#%E9%97%AE%E9%A2%98%E4%B8%89sp%E8%8E%B7%E5%8F%96%E4%B8%8D%E5%88%B0%E5%B1%9E%E6%80%A7)
    - [问题四（属性被过滤）](#%E9%97%AE%E9%A2%98%E5%9B%9B%E5%B1%9E%E6%80%A7%E8%A2%AB%E8%BF%87%E6%BB%A4)
  - [附录](#%E9%99%84%E5%BD%95)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP属性释放常见问题



## 利用SP的Session调试页面确认属性释放

在使用CARSI登录后，SP应该可以获取到IdP释放的属性，SP提供了一个Session调试的页面可以用来查看属性释放。以https://sptest.pku.edu.cn/secure （该URL为该SP的受保护资源页面）为例，其session调试页面为https://sptest.pku.edu.cn/Shibboleth.sso/Session

根据[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md)  中关于“CARSI用户属性”的描述，该Session调试页面应该可以看到IdP释放的属性。如正常情况下应返回类似这样的信息：

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

## 原理说明

### SP端配置及获取属性的原理

注意SP端的/etc/shibboleth/attribute-map.xml 这里有一些默认释放的属性，如：

```
<Attribute name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" id="affiliation">
         <AttributeDecoder xsi:type="ScopedAttributeDecoder" caseSensitive="false"/>
</Attribute>
<Attribute name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" id="affiliation">
         <AttributeDecoder xsi:type="ScopedAttributeDecoder" caseSensitive="false"/>
</Attribute>
```

比如这两个属性，name为从IdP拿到的属性（可以看到包含命名空间），id为映射到本地的属性，即上面通过Session接口查询出来返回的属性affiliation。

注意该文件默认无需修改即可工作于CARSI环境，但该文件是属性映射的重要配置文件，建议理解一下该文件，其中用不到的属性可以移除（但根据[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md)  中关于“CARSI用户属性”的描述，IdP只会释放其中规定的属性，因此不移除其余的属性也没问题）。

如果希望了解多一点，可以看下SP的日志（需要修改/etc/shibboleth/shibd.logger，将最开头的日志级别由INFO改为DEBUG并重启shibd），可以理解属性的来源（这里只贴一部分，建议完整阅读一次登录的日志，便于理解属性释放的完整流程）：

```
2019-07-31 18:22:00 DEBUG Shibboleth.SSO.SAML2 [4] [default]: SSO profile processing completed successfully
2019-07-31 18:22:00 DEBUG Shibboleth.SSO.SAML2 [4] [default]: extracting pushed attributes...
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeExtractor.XML [4] [default]: unable to extract attributes, unknown XML object type: saml2p:Response
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeExtractor.XML [4] [default]: skipping unmapped NameID with format (urn:oasis:names:tc:SAML:2.0:nameid-format:transient)
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeExtractor.XML [4] [default]: unable to extract attributes, unknown XML object type: saml2:AuthnStatement
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeDecoder.Scoped [4] [default]: decoding ScopedAttribute (affiliation) from SAML 2 Attribute (urn:oid:1.3.6.1.4.1.5923.1.1.1.9) with 1 value(s)
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeFilter [4] [default]: filtering 1 attribute(s) from (https://casidp.pku.edu.cn/idp/shibboleth)
2019-07-31 18:22:00 DEBUG Shibboleth.AttributeFilter [4] [default]: applying filtering rule(s) for attribute (affiliation) from (https://casidp.pku.edu.cn/idp/shibboleth)
2019-07-31 18:22:00 DEBUG Shibboleth.SessionCache [4] [default]: creating new session
2019-07-31 18:22:00 DEBUG Shibboleth.SessionCache [4] [default]: storing new session...
```

其中的： applying filtering rule(s) for attribute(affiliation) from (https://casidp.pku.edu.cn/idp/shibboleth) 即为从SP端观察到的属性释放的重要日志。

### IdP端配置及释放属性的原理

有两个关键配置文件。

1、首先关注：/opt/shibboleth-idp/conf/attribute-resolver.xml，这里以IdP+CAS为例

```
<AttributeDefinition id="eduPersonScopedAffiliation" xsi:type="Scoped" scope="%{idp.scope}" sourceAttributeID="employeetype">
         <Dependency ref="employeetype" />
         <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
</AttributeDefinition>
<AttributeDefinition xsi:type="SubjectDerivedAttribute" id="employeetype" principalAttributeName="eduPersonScopedAffiliation">
</AttributeDefinition>
```

这里先看第二个属性id="employeetype"，我们的CAS Server上配置的属性释放为eduPersonScopedAffiliation （参考[CARSI_IdP接入本地认证系统及属性释放.md](CARSI_IdP接入本地认证系统及属性释放.md) ），其值为文本类型（faculty, student, staff, alum, member, affiliate, employee, other之一），这里的配置是先取得它的值。然后再看第一属性id="eduPersonScopedAffiliation"，在id="employeetype"属性之上通过Scoped的属性加上学校域名的后缀，最终由id为eduPersonScopedAffiliation的属性释放。

可是我们SP端读的却是name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation"和name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9"，这又是在哪里映射的呢？可以看到，我们为最外层属性id="eduPersonScopedAffiliation"定义的两个&lt;AttributeEncoder&gt;就是用来干这个的了，这里的name值必须与之前看到的SP的attribute-map.xml完全匹配才可以。

2、另外一个关键配置文件是/opt/shibboleth-idp/conf/attribute-filter.xml

```
<AttributeFilterPolicy id="example1">
         <PolicyRequirementRule xsi:type="ANY" />
         <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
</AttributeFilterPolicy>
```

这里面没什么特别的，就是控制最终释放一个eduPersonScopedAffiliation属性，但是注意这里的attributeID的值是与上面讲的attribute-resolver.xml中的id字段匹配上的。

如果希望了解多一点，可以看下IdP的日志（需要修改/opt/shibboleth-idp/conf/logback.xml，将idp.loglevel.idp的日志级别由INFO改为DEBUG并运行IdP的build.sh重新生成、重启tomcat及httpd），可以理解属性的释放过程（这里只贴一部分，建议完整阅读一次登录的日志，便于理解属性释放的完整流程）：

```
2019-07-31 18:21:58,495 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.resolver.impl.AttributeResolverImpl:252] - Attribute Resolver 'ShibbolethAttributeResolver': Attempting to resolve the following attribute definitions [eduPersonScopedAffiliation, employeetype]
......
2019-07-31 18:21:58,497 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.resolver.impl.AttributeResolverImpl:461] - Attribute Resolver 'ShibbolethAttributeResolver': Finished resolving dependencies for 'eduPersonScopedAffiliation'
......
2019-07-31 18:21:58,500 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.filter.impl.AttributeFilterImpl:125] - Attribute filtering engine 'ShibbolethAttributeFilter'  Beginning process of filtering the following 2 attributes: [eduPersonScopedAffiliation, employeetype]
......
2019-07-31 18:21:58,501 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.filter.AttributeFilterPolicy:162] - Attribute Filter Policy 'example1'  Applying attribute filter policy to current set of attributes: [eduPersonScopedAffiliation, employeetype]
......
2019-07-31 18:21:58,502 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.filter.AttributeRule:168] - Attribute filtering engine '/AttributeFilterPolicyGroup:ShibbolethFilterPolicy/AttributeRule:_cf4ceb60bad32f24054837ae9618c3d8'  Filtering values for attribute 'eduPersonScopedAffiliation' which currently contains 1 values
2019-07-31 18:21:58,502 - 162.105.126.239 - DEBUG [net.shibboleth.idp.attribute.filter.AttributeRule:177] - Attribute filtering engine '/AttributeFilterPolicyGroup:ShibbolethFilterPolicy/AttributeRule:_cf4ceb60bad32f24054837ae9618c3d8'  Filter has permitted the release of 1 values for attribute 'eduPersonScopedAffiliation'
......
2019-07-31 18:21:59,933 - 162.105.126.239 - DEBUG [net.shibboleth.idp.consent.flow.ar.impl.ReleaseAttributes:67] - Profile Action ReleaseAttributes: Attributes before release '{eduPersonScopedAffiliation=IdPAttribute{id=eduPersonScopedAffiliation, displayNames={}, displayDescriptions={}, encoders=[net.shibboleth.idp.saml.attribute.encoding.impl.SAML2ScopedStringAttributeEncoder@23ac74, net.shibboleth.idp.saml.attribute.encoding.impl.SAML1ScopedStringAttributeEncoder@bab64ea8], values=[ScopedStringAttributeValue{value=faculty, scope=pku.edu.cn}]}}'
......
2019-07-31 18:21:59,941 - 162.105.126.239 - DEBUG [net.shibboleth.idp.saml.saml2.profile.impl.AddAttributeStatementToAssertion:187] - Profile Action AddAttributeStatementToAssertion: Encoding attribute eduPersonScopedAffiliation as a SAML 2 Attribute
2019-07-31 18:21:59,941 - 162.105.126.239 - DEBUG [net.shibboleth.idp.saml.attribute.encoding.AbstractSAMLAttributeEncoder:154] - Beginning to encode attribute eduPersonScopedAffiliation
2019-07-31 18:21:59,941 - 162.105.126.239 - DEBUG [net.shibboleth.idp.saml.attribute.encoding.SAMLEncoderSupport:73] - Encoding value faculty@pku.edu.cn of attribute eduPersonScopedAffiliation
2019-07-31 18:21:59,942 - 162.105.126.239 - DEBUG [net.shibboleth.idp.saml.attribute.encoding.AbstractSAMLAttributeEncoder:191] - Completed encoding 1 values for attribute eduPersonScopedAffiliation
2019-07-31 18:21:59,942 - 162.105.126.239 - DEBUG [net.shibboleth.idp.saml.saml2.profile.impl.AddAttributeStatementToAssertion:116] - Profile Action AddAttributeStatementToAssertion: Adding constructed AttributeStatement to Assertion _c94991d0173135fdb5dc5fc23b36654b
```

其中的：Completed encoding 1 values for attribute eduPersonScopedAffiliation 即为从IdP端观察到的属性释放的重要日志，到这一步基本可以说明属性成功释放出来了。

## 常见问题

### 问题一（无Session）

如果Session调试页面显示：

```
A valid session was not found.
```

说明没有登录成功的session，登陆一下即可。

### 问题二（看不到具体的值）

如果Session调试页面显示有属性释放，但是没有具体的值，如affiliation: 1 value(s) 这样的内容：

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

这是因为SP默认配置成了不显示具体值。在SP上可以可以改配置/etc/shibboleth/shibboleth2.xml 来显示具体的值。showAttributeValues改为true：

```
<Handler type="Session" Location="/Session" showAttributeValues="true"/>
```

请注意，出于安全考虑请勿在线上SP开启该功能。CARSI的线上环境已经禁用了该功能。看到有属性值释放即可。

### 问题三（SP获取不到属性）

参考下面的原理说明。

 SP端获取不到属性。IdP端看到日志如下：

```
2019-07-31 18:57:33,223 - 162.105.126.239 - DEBUG [net.shibboleth.idp.consent.flow.ar.impl.ReleaseAttributes:67] - Profile Action ReleaseAttributes: Attributes before release '{eduPersonScopedAffiliation=IdPAttribute{id=eduPersonScopedAffiliation, displayNames={}, displayDescriptions={}, encoders=[net.shibboleth.idp.saml.attribute.encoding.impl.SAML2ScopedStringAttributeEncoder@23ac74, net.shibboleth.idp.saml.attribute.encoding.impl.SAML1ScopedStringAttributeEncoder@bab64ea8], values=[StringAttributeValue{value=faculty@pku.edu.cn}]}}'
......
2019-07-31 18:57:33,248 - 162.105.126.239 - WARN [net.shibboleth.idp.saml.attribute.encoding.AbstractSAMLAttributeEncoder:173] - Skipping value of attribute 'eduPersonScopedAffiliation'; Type net.shibboleth.idp.attribute.StringAttributeValue cannot be encoded by this encoder (net.shibboleth.idp.saml.attribute.encoding.impl.SAML2ScopedStringAttributeEncoder).
```

 重点关注其中的：Type net.shibboleth.idp.attribute.StringAttributeValue cannot be encoded by this encoder
(net.shibboleth.idp.saml.attribute.encoding.impl.SAML2ScopedStringAttributeEncoder

/opt/shibboleth-idp/conf/attribute-resolver.xml配置：

```
<AttributeDefinition xsi:type="SubjectDerivedAttribute" id="eduPersonScopedAffiliation" principalAttributeName="eduPersonScopedAffiliation">
         <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
</AttributeDefinition>
```

这是因为该IdP是采用了IdP+OAuth的配置，OAuth服务器在属性释放的地方直接作为一个字符串释放了eduPersonScopedAffiliation。而不是像上面CAS服务器那样通过Scoped字符串释放的，对比IdP+CAS的日志：

```
2019-07-31 19:15:11,377 - 162.105.126.239 -
DEBUG [net.shibboleth.idp.consent.flow.ar.impl.ReleaseAttributes:67] - Profile
Action ReleaseAttributes: Attributes before release
'{eduPersonScopedAffiliation=IdPAttribute{id=eduPersonScopedAffiliation,
displayNames={}, displayDescriptions={},
encoders=[net.shibboleth.idp.saml.attribute.encoding.impl.SAML2ScopedStringAttributeEncoder@23ac74,
net.shibboleth.idp.saml.attribute.encoding.impl.SAML1ScopedStringAttributeEncoder@bab64ea8],
values=[ScopedStringAttributeValue{value=faculty,
scope=pku.edu.cn}]}}'
```

出错的（IdP+OAuth）：values=[StringAttributeValue{value=faculty@pku.edu.cn}]

成功的（IdP+CAS）：values=[ScopedStringAttributeValue{value=faculty, scope=pku.edu.cn}]

解决办法：将attribute-resolver.xml中的xsi:type="SAML1ScopedString" 改为xsi:type="SAML1String" 即可。

### 问题四（属性被过滤）

![CARSI](/CARSI_IdP属性释放常见问题.files/001.png)

![CARSI](/CARSI_IdP属性释放常见问题.files/002.png)

如上两张图，属性在IdP端释放了（通过IdP的日志查看也确实释放了，没有上面的问题提到的Encoder的问题），但是SP端获取不到属性。查看SP端日志发现：

```
2019-09-18 13:54:59 DEBUG Shibboleth.AttributeFilter [33] [default]: filtering 2 attribute(s) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 DEBUG Shibboleth.AttributeFilter [33] [default]: applying filtering rule(s) for attribute (eppn) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 WARN Shibboleth.AttributeFilter [33] [default]: removed value at position (0) of attribute (eppn) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 DEBUG Shibboleth.AttributeFilter [33] [default]: applying filtering rule(s) for attribute (affiliation) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 WARN Shibboleth.AttributeFilter [33] [default]: removed value at position (0) of attribute (affiliation) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 WARN Shibboleth.AttributeFilter [33] [default]: no values left, removing attribute (eppn) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 WARN Shibboleth.AttributeFilter [33] [default]: no values left, removing attribute (affiliation) from (https://idp.lib.bnu.edu.cn/idp/shibboleth)
2019-09-18 13:54:59 DEBUG Shibboleth.SessionCache [33] [default]: creating new session
2019-09-18 13:54:59 DEBUG Shibboleth.SessionCache [33] [default]: storing new session...
```

 重点关注：removed value at position (0) of attribute(affiliation) from (https://idp.lib.bnu.edu.cn/idp/shibboleth

原因是IdP端对属性释放做了Scope（域名）的限制，具体可参考：https://spaces.at.internet2.edu/display/InCFederation/Scope+in+Metadata

解决：

修改/opt/shibboleth-idp/metadata/idp-metadata.xml，将其中的

```
<shibmd:Scope regexp="false">lib.bnu.edu.cn</shibmd:Scope>
#改为：
<shibmd:Scope regexp="false">bnu.edu.cn</shibmd:Scope>
```

然后重启IdP服务，**并将更新后的metadata上传到联盟**！！

## 附录

文中提到的完整日志文件：

[idp+cas.log](idp+cas.log)

[idp+oauth.log](idp+oauth.log)

[sp_idp+cas.log](sp_idp+cas.log)
