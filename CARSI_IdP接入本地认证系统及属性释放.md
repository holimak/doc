<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [IdP接入本地认证系统及属性释放](#idp%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F%E5%8F%8A%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE)
  - [1. 用户属性定义](#1-%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7%E5%AE%9A%E4%B9%89)
    - [1.1 CARSI用户属性](#11-carsi%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7)
    - [1.2 IdP端配置属性](#12-idp%E7%AB%AF%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7)
  - [2. 接入本地认证系统](#2-%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F)
    - [2.1 接入LDAP认证](#21-%E6%8E%A5%E5%85%A5ldap%E8%AE%A4%E8%AF%81)
    - [2.2 接入OAuth2认证](#22-%E6%8E%A5%E5%85%A5oauth2%E8%AE%A4%E8%AF%81)
    - [2.3 接入CAS认证](#23-%E6%8E%A5%E5%85%A5cas%E8%AE%A4%E8%AF%81)
  - [3. 释放用户属性](#3-%E9%87%8A%E6%94%BE%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7)
  - [4. 用户登录页面添加/取消隐私保护功能](#4-%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2%E6%B7%BB%E5%8A%A0%E5%8F%96%E6%B6%88%E9%9A%90%E7%A7%81%E4%BF%9D%E6%8A%A4%E5%8A%9F%E8%83%BD)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP接入本地认证系统及属性释放



## 1. 用户属性定义

### 1.1 CARSI用户属性

根据 [“CARSI资源共享服务属性要求”](https://www.carsi.edu.cn/docs/attribute_profile_zh.pdf)（2019年9月版本），CARSI IdP目前支持释放两个属性：

- eduPersonScopedAffiliation：核心属性，标识用户的身份，取值为：faculty, student, staff, alum, member, affiliate, employee, other，后面加上@xxx.edu.cn。对应的身份分别为：教师、学生、员工、校友、成员、附属人员，聘用人员、其他。

- eduPersonPrincipalName：推荐属性，人员网络标识，用于机构间认证，格式为user@scope，例如userid@xxx.edu.cn，userid建议使用学校用户管理系统中唯一用户标识，xxx.edu.cn是userid所在的学校。

### 1.2 IdP端配置属性

IdP有两个配置文件，与属性配置有关。

- /opt/shibboleth-idp/conf/attribute-resolver.xml。定义属性。其中的&lt;AttibuteDefinition&gt;定义一个属性，&lt;InputDataConnector&gt;指定属性来源，根据本校情况进行修改，可对接LDAP、OAuth、CAS等。
- /opt/shibboleth-idp/conf/attribute-filter.xml。配置属性释放原则。

下文“接入本地认证系统”分别介绍了LDAP、OAuth、CAS三种本地认证服务的接入方法，其中attribute-resolver.xml的配置略有区别。下文“释放用户属性”介绍了attribute-filter.xml的配置方法，无论使用哪种本地认证服务，其配置都是相同的。

## 2. 接入本地认证系统

请提前下载好本文档提及的相关[附件](https://mgmt.carsi.edu.cn/frontend/web/docs/carsi-idp-installation-manual.zip)。

### 2.1 接入LDAP认证

LDAP认证配置

```
[root@www ~]# vi /opt/shibboleth-idp/conf/ldap.properties
idp.authn.LDAP.authenticator = bindSearchAuthenticator
idp.authn.LDAP.ldapURL = ldap://ldap服务器地址
idp.authn.LDAP.useStartTLS  = false
idp.authn.LDAP.sslConfig = jvmTrust
idp.authn.LDAP.baseDN = ou=People,dc=Test,dc=Test
idp.authn.LDAP.bindDN = cn=Manager,dc=Test,dc=Test
idp.authn.LDAP.bindDNCredential = 密码
idp.authn.LDAP.dnFormat = uid=%s, ou=People,dc=Test,dc=Test
idp.authn.LDAP.subtreeSearch = true
idp.authn.LDAP.userFilter = (uid={user})
idp.attribute.resolver.LDAP.searchFilter = (uid=$resolutionContext.principal)
```

以上属性在ldap.properties已经定义好了，只需要修改属性内容就行，请勿直接拷贝到ldap.properties，会造成属性重复定义，其他未提到的属性按照ldap.properties默认配置就行

因LDAP服务器配置差异，以上配置请和LDAP管理员确认。强烈建议先在IdP所在的服务器上测试一下对本校LDAP的连接性。

如果ldap中不是用uid来唯一区别用户，需要将以下三项配置中的uid改成用来区分用户的属性名称

```
idp.authn.LDAP.dnFormat = uid=%s, ou=People,dc=Test,dc=Test
idp.authn.LDAP.userFilter = (uid={user})
idp.attribute.resolver.LDAP.searchFilter = (uid=$resolutionContext.principal)
```

**注：**华东师范大学冯骐老师分享了一个轻量级的 LDAP测试工具 https://github.com/shanghai-edu/ldap-test-tool ， 可以用来进行测试

配置属性释放，用以下内容替换/opt/shibboleth-idp/conf/attribute-resolver.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<AttributeResolver
        xmlns="urn:mace:shibboleth:2.0:resolver"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">

   <AttributeDefinition xsi:type="ScriptedAttribute" id="eduPersonScopedAffiliation">
        <InputDataConnector ref="myLDAP" attributeNames="usertype"/>
        <Script><![CDATA[
        var localpart = "";
        if(usertype.getValues().get(0)=="staf") localpart = "staff";
        else if(usertype.getValues().get(0)=="std") localpart = "student";
        else localpart = "other";
        eduPersonScopedAffiliation.addValue(localpart + "@%{idp.scope}");
            ]]></Script>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
    </AttributeDefinition>

   <AttributeDefinition xsi:type="Scoped" id="eduPersonPrincipalName" scope="%{idp.scope}">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
        <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eduPersonPrincipalName" encodeType="false" />
   </AttributeDefinition>

   <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
    ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"
    baseDN="%{idp.attribute.resolver.LDAP.baseDN}" 
    principal="%{idp.attribute.resolver.LDAP.bindDN}"
    principalCredential="%{idp.attribute.resolver.LDAP.bindDNCredential}"
    connectTimeout="%{idp.attribute.resolver.LDAP.connectTimeout}"
    responseTimeout="%{idp.attribute.resolver.LDAP.responseTimeout}">
    <FilterTemplate>
        <![CDATA[
            %{idp.attribute.resolver.LDAP.searchFilter}
        ]]>
    </FilterTemplate>
    <ConnectionPool
        minPoolSize="%{idp.pool.LDAP.minSize:3}"
        maxPoolSize="%{idp.pool.LDAP.maxSize:10}"
        blockWaitTime="%{idp.pool.LDAP.blockWaitTime:PT3S}"
        validatePeriodically="%{idp.pool.LDAP.validatePeriodically:true}"
        validateTimerPeriod="%{idp.pool.LDAP.validatePeriod:PT5M}"
        expirationTime="%{idp.pool.LDAP.idleTime:PT10M}"
        failFastInitialize="%{idp.pool.LDAP.failFastInitialize:false}" />
   </DataConnector>

</AttributeResolver>
```

这里，从LDAP读取的属性名称为usertype，如果其值为staf则将eduPersonScopedAffiliation属性设置为staff，如果其值为std则将eduPersonScopedAffiliation属性设置为student。可根据学校LDAP的实际情况进行映射。

### 2.2 接入OAuth2认证

OAuth2认证需要采用标准的授权码（Authorization Code）方式，简单过程如下：

（1）IdP跳转到OAuth2认证页面，用GET方式传递response_type、client_id、state、redirect_uri四个参数；

（2）认证通过后，OAuth2回调到redirect_uri上并带上code参数；

（3）IdP获取code，向OAuth2获取token，用POST方式传递，grant_type=authorization_code、code、client_id、client_secret、redirect_uri；

（4）OAuth2验证后，返回token；

（5）IdP获取token后，向OAuth2请求资源，用POST方式传递token；

（6）OAuth2返回IdP相应的资源。



需要OAuth2管理员提供：

OAuth2认证页面URL、OAuth2用code获取token的URL、OAuth2用token获取资源的URL、IdP认证在OAuth2上的client_id、client_secret

向OAuth2管理员提供：

redirect_uri=https://xxx.xxx.edu.cn/idp/Authn/ExtOauth2?conversation=e1s1



**具体配置：**

（1）新建文件夹并拷贝相关文件：

```
[root@www ~]# mkdir /opt/shibboleth-idp/flows/authn/Shiboauth2
```

把附件中shibcas-authn-beans.xml、shibcas-authn-flow.xml放入文件夹中。

把附件中的no-conversation-state.jsp放入/opt/shibboleth-idp/edit-webapp中。

把附件中的shib-cas-authenticator-3.2.4.jar、cas-client-core-3.4.1.jar放入/opt/shibboleth-idp/edit-webapp/WEB-INF/lib中。

（2）配置web.xml：

```
[root@www ~]# cp /opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml
[root@www ~]# vi /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml
#在<!-- Servlets and servlet mappings -->后加上

<!-- Servlet for receiving a callback from an external CAS Server and continues the IdP login flow -->
     <servlet>
         <servlet-name> ShibOauth2 Auth Servlet</servlet-name>
         <servlet-class>net.unicon.idp.externalauth.ShibcasAuthServlet</servlet-class>
         <load-on-startup>2</load-on-startup>
     </servlet>
     <servlet-mapping>
         <servlet-name> ShibOauth2 Auth Servlet</servlet-name>
         <url-pattern>/Authn/ExtOauth2/*</url-pattern>
     </servlet-mapping>
```

（3）配置idp.properties：

```
[root@www ~]# vi /opt/shibboleth-idp/conf/idp.properties
#修改
idp.authn.flows = Shiboauth2

#新增
shibcas.oauth2UrlPrefix = http://xxx.xxx.xxx.xxx #OAuth2服务器域名
shibcas.oauth2LoginUrl = ${shibcas.oauth2UrlPrefix}/xxx?response_type=code&client_id=xxx&state=xyz #OAuth2认证页面以及需要传递的参数，state可以配置成任意字符串
shibcas.serverName = https://xxx.xxx.xxx.xxx #IdP的域名
shibcas.oauth2TokenUrl = http://xxx.xxx.xxx.xxx/xxx # OAuth2用code获取token的URL
shibcas.oauth2ResourceUrl = http://xxx.xxx.xxx.xxx/xxx # OAuth2用token获取资源的URL
shibcas.oauth2clientid = testclient
shibcas.oauth2clientsecret = testpass
shibcas.oauth2redirecturi = https://xxx.xxx.xxx/idp/Authn/ExtOauth2?conversation=e1s1
```

（4）配置general-authn.xml：

```
[root@www ~]# vi /opt/shibboleth-idp/conf/authn/general-authn.xml
#在<util:list id="shibboleth.AvailableAuthenticationFlows">后新增

<bean id="authn/Shiboauth2" parent="shibboleth.AuthenticationFlow"
                 p:passiveAuthenticationSupported="true"
                 p:forcedAuthenticationSupported="true"
                 p:nonBrowserSupported="false" />
```

（5）配置属性释放，用以下内容替换/opt/shibboleth-idp/conf/attribute-resolver.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<AttributeResolver
         xmlns="urn:mace:shibboleth:2.0:resolver"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="eduPersonScopedAffiliation" principalAttributeName="eduPersonScopedAffiliation">
         <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
   </AttributeDefinition>

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id=" eduPersonPrincipalName" principalAttributeName="eduPersonPrincipalName">
         <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eduPersonPrincipalName" encodeType="false" />
   </AttributeDefinition>

</AttributeResolver>
```

（6）重新编译war文件，并且重启tomcat：

```
[root@www ~]# cd /opt/shibboleth-idp/bin
[root@www ~]# ./build.sh

Installation Directory: [/opt/shibboleth-idp] #enter

Rebuilding /opt/shibboleth-idp/war/idp.war ...

...done

BUILD SUCCESSFUL

Total time: 3 seconds

[root@www ~]# systemctl restart tomcat
```

**备注：关于OAuth服务器端配置：**

这里我们使用开源的[oauth2-server-php](http://bshaffer.github.io/oauth2-server-php-docs/cookbook/)进行搭建的OAuth服务器。需要OAuth2服务器在获取资源的地方以json格式返回eduPersonScopedAffiliation和eduPersonPrincipalName属性信息，涉及到的文件就是resource.php，如下代码所示：最后一行代码以json形式返回了需要的属性，这里只为展示使用了固定的值，学校可根据实际情况读取LDAP或JDBC数据库中的数据进行返回。

```
<?php
// include our OAuth2 Server object
require_once __DIR__.'/server.php';

// Handle a request to a resource and authenticate the access token
if (!$server->verifyResourceRequest(OAuth2\Request::createFromGlobals())) {
     $server->getResponse()->send();
     die;
}

echo json_encode(array('success' => 'true', 'uid' => '20190606', 'eduPersonScopedAffiliation' => 'faculty@pku.edu.cn', 'eduPersonPrincipalName' => '20190606@pku.edu.cn')); 
```



### 2.3 接入CAS认证

具体配置：

（1）环境配置：

下载并解压shib-cas-authn3-3.2.3.zip（已作为附件提供，也可自行[下载](https://github.com/Unicon/shib-cas-authn3)）

解压后将IDP_HOME/flows/authn/Shibcas目录下的2个XML文件shibcas-authn-beans.xml和shibcas-authn-flow.xml拷贝至IdP服务器的/opt/shibboleth-idp/flow/authn/Shibcas/下。

将IDP_HOME/edit-webapp/目录下的no-conversation-state.jsp文件拷贝至IdP服务器的/opt/shibboleth-idp/edit-webapp/下。

将文件cas-client-core-3.4.1.jar和shib-cas-authenticator-3.2.3.jar拷贝至IdP服务器的/opt/shibboleth-idp/edit-webapp/WEB-INF/lib/下。

（2）配置web.xml：

```
[root@www ~]# cp /opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml
[root@www ~]# vi /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml

#在<!-- Servlets and servlet mappings -->后加上

<!-- Servlet for receiving a callback from an external CAS Server and continues the IdP login flow -->
     <servlet>
         <servlet-name>ShibCas Auth Servlet</servlet-name>
         <servlet-class>net.unicon.idp.externalauth.ShibcasAuthServlet</servlet-class>
         <load-on-startup>2</load-on-startup>
     </servlet>
     <servlet-mapping>
         <servlet-name>ShibCas Auth Servlet</servlet-name>
         <url-pattern>/Authn/ExtCas/*</url-pattern>
</servlet-mapping>
```

（3）配置idp.properties：

```
[root@www ~]# vi /opt/shibboleth-idp/conf/idp.properties
#修改
idp.authn.flows = Shibcas

#新增
shibcas.casServerUrlPrefix = http://xxx.xxx.xxx.xxx/cas  #CAS服务器域名
shibcas.casServerLoginUrl = ${shibcas.casServerUrlPrefix}/login
shibcas.serverName = https://xxx.xxx.xxx.xxx #IdP的域名  
```

（4）配置general-authn.xml：

```
[root@www ~]# vi /opt/shibboleth-idp/conf/authn/general-authn.xml
\#在<util:list id="shibboleth.AvailableAuthenticationFlows">后新增

<bean id="authn/Shibcas" parent="shibboleth.AuthenticationFlow"
                 p:passiveAuthenticationSupported="true"
                 p:forcedAuthenticationSupported="true"
                 p:nonBrowserSupported="false" />
```

（5）配置属性释放，用以下内容替换/opt/shibboleth-idp/conf/attribute-resolver.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<AttributeResolver
         xmlns="urn:mace:shibboleth:2.0:resolver"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">

     <AttributeDefinition id="eduPersonScopedAffiliation" xsi:type="Scoped" scope="%{idp.scope}" sourceAttributeID="employeetype">
         <Dependency ref="employeetype" />
         <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
     </AttributeDefinition>

     <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="employeetype" principalAttributeName="eduPersonScopedAffiliation"></AttributeDefinition>
     
     <AttributeDefinition id="eduPersonPrincipalName" xsi:type="Scoped" scope="%{idp.scope}" sourceAttributeID="employeename">
         <Dependency ref="employeename" />
         <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eduPersonPrincipalName" encodeType="false" />
     </AttributeDefinition>

     <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="employeename" principalAttributeName="eduPersonPrincipalName"></AttributeDefinition>

</AttributeResolver>
```

说明：eduPersonScopedAffiliation为CAS直接释放的属性，其值为文本类型（faculty, student, staff, alum, member, affiliate, employee, other之一）。这里的配置是先取得它的值（存入employeetype属性），再在其上通过Scoped的属性加上学校域名的后缀，最终由id为eduPersonScopedAffiliation的属性释放。eduPersonPrincipalName属性同理进行了配置。

（5）重新编译war文件，并且重启tomcat：

```
[root@www ~]# cd /opt/shibboleth-idp/bin
[root@www ~]# ./build.sh

Installation Directory: [/opt/shibboleth-idp] #enter

Rebuilding /opt/shibboleth-idp/war/idp.war ...

...done

BUILD SUCCESSFUL

Total time: 3 seconds

[root@www ~]# systemctl restart tomcat
```

**备注：关于CAS服务器端配置：**

以CAS 6.0.x版本对接LDAP为例。参考[官方文档](https://apereo.github.io/2018/11/16/cas60-gettingstarted-overlay/)配置、安装CAS服务器。须配置application.properties文件（以不使用ssl为例，须根据学校具体配置进行相应调整各属性）：

```
cas.authn.ldap[0].type=AUTHENTICATED
cas.authn.ldap[0].ldapUrl=ldap://xx.xx.xx.xx:389
cas.authn.ldap[0].useSsl=false
cas.authn.ldap[0].baseDn=ou=People,dc=pku,dc=edu
cas.authn.ldap[0].searchFilter=uid={user}
cas.authn.ldap[0].bindDn=cn=Manager,dc=pku,dc=edu
cas.authn.ldap[0].bindCredential=xxxxxxxx
cas.authn.ldap[0].principalAttributeList=employeeType:eduPersonScopedAffiliation,uid:eduPersonPrincipalName
cas.authn.attributeRepository.defaultAttributesToRelease=eduPersonScopedAffiliation,eduPersonPrincipalName
```

这里的环境，代表用户身份的字段为employeeType，最后两行的配置是将其作为eduPersonScopedAffiliation属性释放给IdP。eduPersonPrincipalName同理进行了配置。可根据学校具体配置进行相应调整。



## 3. 释放用户属性

使用以下内容替换/opt/shibboleth-idp/conf/attribute-filter.xml，配置属性释放原则：
#xsi:type=”ANY”表示的是对任意SP释放属性，permitAny=”true”，表示的是释放任意eduPersonScopedAffiliation取值的属性
```
<?xml version="1.0" encoding="UTF-8"?>
<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
         xmlns="urn:mace:shibboleth:2.0:afp"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

     <AttributeFilterPolicy id="carsiAttrFilterPolicy">
         <PolicyRequirementRule xsi:type="ANY" />
         <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
         <AttributeRule attributeID="eduPersonPrincipalName" permitAny="true" />
     </AttributeFilterPolicy>

</AttributeFilterPolicyGroup>
```

如果仅对某个或者某几个SP释放属性：

```
#把<PolicyRequirementRule xsi:type="ANY" />改为

<PolicyRequirementRule xsi:type="Requester" value="https://sp.example.org" />

#https://sp.example.org为SP的entityID，表示只对sp.example.org释放属性

<PolicyRequirementRule xsi:type="OR">
     <Rule xsi:type="Requester" value="https://sp.example.org" />
     <Rule xsi:type="Requester" value="https://another.example.org/shibboleth" />
</PolicyRequirementRule>

#表示对sp.example.org 或者 another.example.org释放属性
```

如果对释放属性取值做限制：

```
#把<AttributeRule attributeID="eduPersonPrincipalName" permitAny="true" />改为

<AttributeRule attributeID="eduPersonPrincipalName">
     <PermitValueRule xsi:type="Value" value="jsmith" ignoreCase="true" />
</AttributeRule>

#value="jsmith"表示只释放eduPersonPrincipalName值为jsmith的属性
```
如果需要对释放属性限制多个取值:
```
#把<AttributeRule attributeID="eduPersonPrincipalName" permitAny="true" />改为

<AttributeRule attributeID="eduPersonPrincipalName">
  <PermitValueRule xsi:type="OR">
     <Rule xsi:type="Value" value="jsmith" ignoreCase="true" />
     <Rule xsi:type="Value" value="jimmy" ignoreCase="true" />
  </PermitValueRule>
</AttributeRule>

#value="jsmith" value="jimmy"表示只释放eduPersonPrincipalName值为jsmith或jimmy的属性
```

## 4. 用户登录页面添加/取消隐私保护功能

用户登录，输入用户名密码后，可允许用户参与个人隐私保护和属性释放过程，包括“隐私保护须知”页面和“属性释放选择”页面。如开放给用户，需进行如下配置。如无需添加用户隐私保护功能，可跳过此步骤。

配置用户是否可见隐私保护须知页面，包括三个配置文件：messages.properties、consent-intercept-config.xml、relying-party.xml。配置过程如下：

- 配置/opt/shibboleth-idp/messages/messages.properties文件：

```
#提示用户隐私保护须知。不配置以下参数，该须知用户不可见。
my-terms = my-tou
my-tou.title = Example Terms of Use
my-tou.text = This is an example Terms of Use
```

- 配置/opt/shibboleth-idp/conf/intercept/consent-intercept-config.xml文件

```
#注释
<alias alias="shibboleth.consent.terms-of-use.Key" name="shibboleth.RelyingPartyIdLookup.Simple" />    

#添加
<bean id="shibboleth.consent.terms-of-use.Key" class="com.google.common.base.Functions" factory-method="constant">
<constructor-arg value="my-terms"/>
</bean>
```

- 配置/opt/shibboleth-idp/conf/relying-party.xml文件

```
<bean parent="SAML2.SSO" p:postAuthenticationFlows="attribute-release"/>
#替换成
<bean parent="SAML2.SSO" p:postAuthenticationFlows="#{ {'terms-of-use', 'attribute-release'} }" />
```

配置用户是否可见属性释放选择页面（默认是可见的），如果要关闭，配置/opt/shibboleth-idp/conf/relying-party.xml文件

```
<bean parent="Shibboleth.SSO" p:postAuthenticationFlows="attribute-release" />
<bean parent="SAML2.SSO" p:postAuthenticationFlows="#{ {'terms-of-use', 'attribute-release'} }" />

#替换成

<bean parent="Shibboleth.SSO" />
<bean parent="SAML2.SSO" />
```

- 在用户可见属性释放选择页面的前提下，可以配置用户是否可以选择释放哪个属性

```
[root@www ~]# vi /opt/shibboleth-idp/conf/idp.properties
#true：用户可见释放属性页面；false或者注释掉该行：用户不可见属性释放页面。
idp.consent.allowPerAttribute=true
```

