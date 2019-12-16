此文档在CentOS7上验证通过。

- eduPersonTargetedID（ePTID）：是一个永久的，可读性不强的身份识别码，用于唯一标识用户身份，同一个IdP的同一个用户为不同的SP提供不同的ePTID，在保护用户隐私的前提下支持SP区分用户。
- eduPersonEntitlement（ePE）：标识用户访问特定资源的权限的URI。表示用户有权限访问此资源。取值固定为“urn:mace:dir:entitlement:common-lib-terms”。表示参照IdP和图书馆类SP已经达成的线下协议，IdP用户去访问SP资源。建议将此属性释放给指定SP。
- eduPersonPrincipalName（ePPN）：用于标识用户身份，取值为user@scope。为了保护用户隐私，2019年12月起，建议IdP保留ePPN定义，取消ePPN释放，改为释放ePTID。

#### 配置eduPersonEntitlement

属性定义

```
[root@www ~]# vi attribute-resolver.xml
```

新增

```
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
```

属性释放

```
[root@www ~]# vi attribute-filter.xml
```

释放给某个SP，将下述 https://sp.example.org 替换成SP的entityID，比如Elsevier的entityID

```
<AttributeFilterPolicy id="carsiAttrFilterToSPPolicy">
     <PolicyRequirementRule xsi:type="Requester" value="https://sp.example.org" />
     <AttributeRule attributeID="eduPersonEntitlement" permitAny="true" />
</AttributeFilterPolicy>
```

•	重启tomcat

```
[root@www ~]#systemctl restart tomcat
```

##### 配置eduPersonTargetedID

共两种配置方式。一种是通过数据库永久存放用户ePTID，另一种是依据一定的算法每次计算用户的ePTID。选用一种即可。

##### 第一种方式：采用数据库永久存放用户ePTID

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

##### 第二种方式：依据一定的算法每次计算用户的ePTID，释放给所有SP

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

##### 去掉eduPersonPrincipalName属性

取消属性释放，其他SP无法再获得此属性数据。

```
[root@www ~]#vi attribute-filter.xml
```

删掉

```
<AttributeRule attributeID="eduPersonPrincipalName" permitAny="true" />
```

•	重启tomcat

```
[root@www ~]#systemctl restart tomcat
```

##### 完整配置文件参考

attribute-resolver.xml（数据库方式配置ePTID）

```
<?xml version="1.0" encoding="UTF-8"?>

<AttributeResolver
 xmlns="urn:mace:shibboleth:2.0:resolver"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">


 <AttributeDefinition xsi:type="Scoped" id="eduPersonScopedAffiliation" scope="%{idp.scope}">
 <InputDataConnector ref="myLDAP" attributeNames="employeeType"/>
 <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
 <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
 </AttributeDefinition>

<AttributeDefinition xsi:type="Scoped" id="eduPersonPrincipalName" scope="%{idp.scope}">
<InputDataConnector ref="myLDAP" attributeNames="uid"/>
<AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonPrincipalName" encodeType="false" />
<AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" friendlyName="eduPersonPrincipalName" encodeType="false" />
</AttributeDefinition>

 
<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
 <InputDataConnector ref="myStoredId" attributeNames="persistentID"/>
 <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
 <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
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

<DataConnector id="myStoredId" xsi:type="StoredId" generatedAttributeID="persistentID" salt="%{idp.persistentId.salt}" queryTimeout="0">
 <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}"/>
 <BeanManagedConnection>MyDataSource</BeanManagedConnection>
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

attribute-resolver.xml（计算方式配置ePTID）

```
<?xml version="1.0" encoding="UTF-8"?>

<AttributeResolver
 xmlns="urn:mace:shibboleth:2.0:resolver"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">


 <AttributeDefinition xsi:type="Scoped" id="eduPersonScopedAffiliation" scope="%{idp.scope}">
 <InputDataConnector ref="myLDAP" attributeNames="employeeType"/>
 <AttributeEncoder xsi:type="SAML1ScopedString" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
 <AttributeEncoder xsi:type="SAML2ScopedString" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
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

 

<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
 <InputDataConnector ref="ComputedIDConnector" attributeNames="computedID"/>
 <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
 <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
 </AttributeDefinition>


 <DataConnector id="ComputedIDConnector" xsi:type="ComputedId" 
 generatedAttributeID="computedID"
 salt="xxxxxxxxxxxxxx"
 encoding="BASE64">
 <InputAttributeDefinition ref="eduPersonPrincipalName" />
 </DataConnector>


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

attribute-filter.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
 xmlns="urn:mace:shibboleth:2.0:afp"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

<AttributeFilterPolicy id="carsiAttrFilterToSPPolicy">
 <PolicyRequirementRule xsi:type="ANY" />
 <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
 <AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
 </AttributeFilterPolicy></AttributeFilterPolicyGroup>
 
 <AttributeFilterPolicy id="carsiAttrFilterToSingleSPPolicy">
     <PolicyRequirementRule xsi:type="Requester" value="https://sp.example.org" />
     <AttributeRule attributeID="eduPersonEntitlement" permitAny="true" />
</AttributeFilterPolicy>
```
