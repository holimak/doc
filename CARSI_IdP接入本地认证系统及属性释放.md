<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CARSI IdP接入本地认证系统及属性释放](#carsi-idp%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F%E5%8F%8A%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE)
  - [1. 用户属性定义](#1-%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7%E5%AE%9A%E4%B9%89)
    - [1.1 CARSI用户属性](#11-carsi%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7)
    - [1.2 IdP端配置属性](#12-idp%E7%AB%AF%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7)
  - [2. 接入本地认证系统](#2-%E6%8E%A5%E5%85%A5%E6%9C%AC%E5%9C%B0%E8%AE%A4%E8%AF%81%E7%B3%BB%E7%BB%9F)
    - [2.1 接入LDAP认证](#21-%E6%8E%A5%E5%85%A5ldap%E8%AE%A4%E8%AF%81)
    - [2.2 接入OAuth2认证](#22-%E6%8E%A5%E5%85%A5oauth2%E8%AE%A4%E8%AF%81)
    - [2.3 接入CAS认证](#23-%E6%8E%A5%E5%85%A5cas%E8%AE%A4%E8%AF%81)
  - [3. 配置eduPersonTargetedID](#3-%E9%85%8D%E7%BD%AEedupersontargetedid)
    - [3.1 第一种方式：采用数据库永久存放用户ePTID](#31-%E7%AC%AC%E4%B8%80%E7%A7%8D%E6%96%B9%E5%BC%8F%E9%87%87%E7%94%A8%E6%95%B0%E6%8D%AE%E5%BA%93%E6%B0%B8%E4%B9%85%E5%AD%98%E6%94%BE%E7%94%A8%E6%88%B7eptid)
    - [3.2 第二种方式：依据一定的算法每次计算用户的ePTID，释放给所有SP](#32-%E7%AC%AC%E4%BA%8C%E7%A7%8D%E6%96%B9%E5%BC%8F%E4%BE%9D%E6%8D%AE%E4%B8%80%E5%AE%9A%E7%9A%84%E7%AE%97%E6%B3%95%E6%AF%8F%E6%AC%A1%E8%AE%A1%E7%AE%97%E7%94%A8%E6%88%B7%E7%9A%84eptid%E9%87%8A%E6%94%BE%E7%BB%99%E6%89%80%E6%9C%89sp)
  - [4. 释放用户属性](#4-%E9%87%8A%E6%94%BE%E7%94%A8%E6%88%B7%E5%B1%9E%E6%80%A7)
  - [5. 用户登录页面添加/取消隐私保护功能](#5-%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2%E6%B7%BB%E5%8A%A0%E5%8F%96%E6%B6%88%E9%9A%90%E7%A7%81%E4%BF%9D%E6%8A%A4%E5%8A%9F%E8%83%BD)
  - [6. 属性释放常见问题](#6-%E5%B1%9E%E6%80%A7%E9%87%8A%E6%94%BE%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CARSI IdP接入本地认证系统及属性释放



## 1. 用户属性定义

### 1.1 CARSI用户属性

根据 [“CARSI资源共享服务属性要求”](https://www.carsi.edu.cn/docs/attribute_profile_zh.pdf)（2019年9月版本），CARSI IdP目前需要定义四个属性，释放三个属性，需要释放的属性：

- eduPersonScopedAffiliation：核心属性，标识用户的身份，取值为：faculty, student, staff, alum, member, affiliate, employee, other，后面加上@xxx.edu.cn。对应的身份分别为：教师、学生、员工、校友、成员、附属人员，聘用人员、其他。
- eduPersonTargetedID（ePTID）：是一个永久的，可读性不强的身份识别码，用于唯一标识用户身份，同一个IdP的同一个用户为不同的SP提供不同的ePTID，在保护用户隐私的前提下支持SP区分用户。
- eduPersonEntitlement（ePE）：标识用户访问特定资源的权限的URI。表示用户有权限访问此资源。取值固定为“urn:mace:dir:entitlement:common-lib-terms”。表示参照IdP和图书馆类SP已经达成的线下协议，IdP用户去访问SP资源。建议将此属性释放给指定SP。

需要定义的属性（仅需定义无需释放，ePTID的释放依赖此属性）：

- eduPersonPrincipalName（ePPN）：用于标识用户身份，取值为user@scope。为了保护用户隐私，2019年12月起，建议IdP保留ePPN定义，取消ePPN释放，改为释放ePTID。

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

以上属性在ldap.properties已经定义好了，只需要修改属性内容即可，请勿直接拷贝到ldap.properties，会造成属性重复定义，其他未提到的属性按照ldap.properties默认配置即可。

如果LDAP中不是用uid来唯一区别用户，需要将以下三项配置中的uid改成用来区分用户的属性名称

```
idp.authn.LDAP.dnFormat = uid=%s, ou=People,dc=Test,dc=Test
idp.authn.LDAP.userFilter = (uid={user})
idp.attribute.resolver.LDAP.searchFilter = (uid=$resolutionContext.principal)
```

**注：**因LDAP服务器配置差异，以上配置请和LDAP管理员确认，需要了解本校LDAP的目录结构以及字段定义后再做修改。强烈建议先在IdP所在的服务器上测试一下对本校LDAP的连接性，以及查看一下从LDAP管理员处要来的属性及其取值是否正确。华东师范大学冯骐老师分享了一个轻量级的 LDAP测试工具 https://github.com/shanghai-edu/ldap-test-tool ， 可以用来进行测试。或使用类似LDAP Browser或LDAP Admin这样的工具进行验证。最好在IdP所在服务器上进行验证，以排除环境问题的影响。

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

<AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
    <InputDataConnector ref="staticAttributes" attributeNames="eduPersonEntitlement" />
    <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonEntitlement" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.7" friendlyName="eduPersonEntitlement" encodeType="false"/>
</AttributeDefinition>

<DataConnector id="staticAttributes" xsi:type="Static">
    <Attribute id="eduPersonEntitlement">
        <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
    </Attribute>
</DataConnector>


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

**注1：**以上配置中关于eduPersonScopedAffiliation属性的取值，这里进行简单说明。通俗点的理解就是LDAP中不同的账号类型标记不同的eduPersonScopedAffiliation属性的取值，比如教工的标记为教工，学生的标记为学生，需要向管理员确认LDAP是通过哪个字段来区分的，以及相应的取值范围。这里的例子，从LDAP读取的属性名称为usertype，如果其值为staf则将eduPersonScopedAffiliation属性设置为staff，如果其值为std则将eduPersonScopedAffiliation属性设置为student。可根据学校LDAP的实际情况进行映射，根据情况修改上面配置样例中的attributeNames="**usertype**"，以及Script块中的判断条件和取值的逻辑。

**注2：**关于eduPersonPrincipalName属性，上面配置样例中是直接通过LDAP属性uid获取的，即attributeNames="uid"。如果LDAP中不是用uid来唯一区别用户，则这里也需要进行相应修改。

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
<AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
    <InputDataConnector ref="staticAttributes" attributeNames="eduPersonEntitlement" />
    <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonEntitlement" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.7" friendlyName="eduPersonEntitlement" encodeType="false"/>
</AttributeDefinition>

<DataConnector id="staticAttributes" xsi:type="Static">
    <Attribute id="eduPersonEntitlement">
        <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
    </Attribute>
</DataConnector>

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

解压后将IDP_HOME/flows/authn/Shibcas目录下的2个XML文件shibcas-authn-beans.xml和shibcas-authn-flow.xml拷贝至IdP服务器的/opt/shibboleth-idp/flows/authn/Shibcas/下。

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

<AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
    <InputDataConnector ref="staticAttributes" attributeNames="eduPersonEntitlement" />
    <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonEntitlement" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.7" friendlyName="eduPersonEntitlement" encodeType="false"/>
</AttributeDefinition>


<DataConnector id="staticAttributes" xsi:type="Static">
    <Attribute id="eduPersonEntitlement">
        <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
    </Attribute>
</DataConnector>

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

## 3. 配置eduPersonTargetedID

共两种配置方式。一种是通过数据库永久存放用户ePTID，另一种是依据一定的算法每次计算用户的ePTID。选用一种即可。

### 3.1 第一种方式：采用数据库永久存放用户ePTID

出于安全考虑，建议数据库只允许本地访问、删除匿名用户、禁止远程登录、删除test数据库。

•	安装数据库，`bind-address=127.0.0.1`只允许本地访问

```
[root@www ~]#yum -y install mariadb mariadb-server
[root@www ~]#vi /etc/my.cnf
add follows within [mysqld] section
[mysqld]
character-set-server=utf8
bind-address=127.0.0.1
[root@www ~]#systemctl start mariadb
[root@www ~]#systemctl enable mariadb
```

•	数据库配置

```
[root@www ~]#mysql_secure_installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

# 回车
Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

# 设置root密码
Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

# 删除匿名账户
Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

# 禁止远程登陆
Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

# 删除test数据库
Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

# 刷新权限
Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

•	安装mysql-connector-java

```
[root@www ~]#yum -y install mysql-connector-java
[root@www ~]#ln -s /usr/share/java/mysql-connector-java.jar /usr/share/tomcat/lib/
```

•	数据库初始化，包括建一个IdP工作账号，建一张idp_db表，用于存放ePTID相关信息。

先用root账号登录，创建用户IdP使用的账户，username为用户名，password为密码

```
[root@www ~]# mysql -u root -p
create user 'username'@'localhost' identified by 'password';
```

切换到创建好的账号，username为刚创建好的账户

```
[root@www ~]# mysql -u username -p
CREATE DATABASE idp_db CHARACTER SET utf8 COLLATE utf8_bin;
use idp_db;
CREATE TABLE shibpid (
    localEntity VARCHAR(255) NOT NULL,
    peerEntity VARCHAR(255) NOT NULL,
    persistentId VARCHAR(50) NOT NULL,
    principalName VARCHAR(50) NOT NULL,
    localId VARCHAR(50) NOT NULL,
    peerProvidedId VARCHAR(50) NULL,
    creationDate TIMESTAMP NOT NULL,
    deactivationDate TIMESTAMP NULL,
    PRIMARY KEY (localEntity, peerEntity, persistentId)
);
```

•	IdP配置，包括：开启NameID生成器自动生成ePTID，定义ePTID属性，将ePTID释放给所有SP。

开启NameID生成器

```
[root@www ~]# vi /opt/shibboleth-idp/conf/saml-nameid.xml
<util:list id="shibboleth.SAML2NameIDGenerators">
    <ref bean="shibboleth.SAML2TransientGenerator" />
    <ref bean="shibboleth.SAML2PersistentGenerator" />
</util:list>
```

配置NameID参数，依赖某个可唯一代表用户的id类属性作为原属性，比如eduPersonPrincipalName、uid等，`idp.persistentId.salt = xxxxxxxxxxxxxxxxxxxx`为加密盐值，长度≥16，改成随机字符串

可以使用openssl命令生成随机字符串

```
openssl rand 32 -base64
```

```
[root@www ~]# vi saml-nameid.properites
idp.persistentId.sourceAttribute = 可唯一代表用户id的属性名，需在attribute-resolver.xml中提前定义
idp.persistentId.salt = xxxxxxxxxxxxxxxxxxxx
idp.persistentId.encoding = BASE64
idp.persistentId.dataSource = MyDataSource
```

属性定义

```
[root@www ~]# vi attribute-resolver.xml
增加
<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
    <InputDataConnector ref="myStoredId" attributeNames="persistentID"/>
    <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
</AttributeDefinition>
<DataConnector id="myStoredId" xsi:type="StoredId" generatedAttributeID="persistentID" salt="%{idp.persistentId.salt}" queryTimeout="0">
    <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}"/>
    <BeanManagedConnection>MyDataSource</BeanManagedConnection>
</DataConnector>
```

属性释放

```
[root@www ~]# vi attributer-filter.xml
```

在`<PolicyRequirementRule xsi:type="ANY">`新增

```
<AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
```

定义IdP和数据库的连接，`p:username`改成数据库用户名，`p:password`改成数据库的密码

```
[root@www ~]# vi global.xml
增加
<bean id="MyDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
p:driverClassName="com.mysql.jdbc.Driver"
p:url="jdbc:mysql://localhost:3306/idp_db"
p:username="username"
p:password="password"    
p:maxIdle="5"
p:maxWaitMillis="15000"
p:testOnBorrow="true"
p:validationQuery="select 1"
p:validationQueryTimeout="5" />
```

•	重启tomcat

```
[root@www ~]#systemctl restart tomcat
```

### 3.2 第二种方式：依据一定的算法每次计算用户的ePTID，释放给所有SP

此种方式依据事先配置好的ePTID生成算法，在每次需属性释放时进行计算，无需配置数据库，方法简单。不重装系统、不修改配置的情况下，同一用户对同一SP的ePTID唯一。

- 属性定义，`salt="xxxxxxxxxxxxxxxxxxxx"`为加密盐值，长度≥16，改成随机字符串

可以使用openssl命令生成随机字符串

```
openssl rand 32 -base64
```

```
<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
    <InputDataConnector ref="ComputedIDConnector" attributeNames="computedID"/>
    <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
</AttributeDefinition> 
<DataConnector id="ComputedIDConnector" xsi:type="ComputedId" generatedAttributeID="computedID" salt="xxxxxxxxxxxxxxxxxxxx" encoding="BASE64">
    <InputAttributeDefinition ref="eduPersonPrincipalName" />
</DataConnector> 
```

`<InputAttributeDefinition ref="eduPersonPrincipalName"/>`为将eduPersonPrincipalName作为生成eduPersonTargetedID的源属性，将IdP对外释放的ePTID属性和eduPersonPrincipalName属性进行关联。

- 属性释放

```
<AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
```

•	重启tomcat

```
[root@www ~]#systemctl restart tomcat
```

## 4. 释放用户属性

使用以下内容替换/opt/shibboleth-idp/conf/attribute-filter.xml，并将下述 https://sp.example.org 替换成SP的entityID，比如Elsevier的entityID，配置属性释放原则： #xsi:type=”ANY”表示的是对任意SP释放属性，permitAny=”true”，表示的是释放任意eduPersonScopedAffiliation取值的属性

```
<?xml version="1.0" encoding="UTF-8"?>
<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
         xmlns="urn:mace:shibboleth:2.0:afp"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

     <AttributeFilterPolicy id="carsiAttrFilterPolicy">
         <PolicyRequirementRule xsi:type="ANY" />
         <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
         <AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
     </AttributeFilterPolicy>
     
     <AttributeFilterPolicy id="carsiAttrFilterToSPPolicy">
         <PolicyRequirementRule xsi:type="Requester" value="https://sp.example.org" />
         <AttributeRule attributeID="eduPersonEntitlement" permitAny="true" />
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

## 5. 用户登录页面添加/取消隐私保护功能

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

## 6. 属性释放常见问题

请参考文档：[CARSI_IdP属性释放常见问题.md](CARSI_IdP属性释放常见问题.md)
