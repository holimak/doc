<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [SP+OAuth网关方式接入SP](#spoauth%E7%BD%91%E5%85%B3%E6%96%B9%E5%BC%8F%E6%8E%A5%E5%85%A5sp)
  - [前言](#%E5%89%8D%E8%A8%80)
  - [CARSI SP+OAuth网关](#carsi-spoauth%E7%BD%91%E5%85%B3)
  - [SP+OAuth应用系统访问流程](#spoauth%E5%BA%94%E7%94%A8%E7%B3%BB%E7%BB%9F%E8%AE%BF%E9%97%AE%E6%B5%81%E7%A8%8B)
  - [应用系统接入步骤](#%E5%BA%94%E7%94%A8%E7%B3%BB%E7%BB%9F%E6%8E%A5%E5%85%A5%E6%AD%A5%E9%AA%A4)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# SP+OAuth网关方式接入SP

## 前言

应用系统接入CARSI资源共享服务之后，可以为CARSI用户提供服务。CARSI联盟目前提供两种应用系统接入技术路线，管理员可酌情选取其中的一种实现。

- 以Shibboleth SP的方式接入应用系统。

这种方式适用于应用系统已经支持Shibboleth认证机制、已实现Shibboleth SP的情况，或适用于希望新建/改建应用系统的认证授权部分以支持Shibboleth SP的情况。此方式相当于自主安装SP 环境来保护待接入的应用系统，并将该SP加入CARSI联盟。

以CARSI会员单位账号登录登陆 [CARSI会员自服务系统](https://mgmt.carsi.edu.cn) 之后，可根据“调试帮助页面”的《CARSI SP 安装配置文档》，安装Shibboleth SP环境、完成与CARSI联盟交换Metadata元数据文件等配置。

- 以SP+OAuth的方式接入应用系统。

通过CARSI联盟提供的SP+OAuth环境，将现有的应用系统以OAuth的方式，接入CARSI SP+OAuth网关，由该网关代表应用系统来支持Shibboleth SP功能，应用系统无需自己搭建SP服务。此方式对应用系统改动小、部署速度快、接入流程简单。要求应用系统支持OAuth 2.0协议。

本文档介绍第2种方案。



## CARSI SP+OAuth网关

CARSI SP+OAuth网关由Shibboleth SP和OAuth Server两部分功能组成。其中的Shibboleth SP代表即将接入的应用系统，支持CARSI用户的访问。OAuth Server采用OAuth 2.0协议与应用系统对接。网关的作用是将来自于应用系统的、以OAuth Client发送的认证请求，通过OAuth Server和Shibboleth SP，转发给CARSI联盟，在用户自己所在学校完成身份认证之后，将认证结果通过本网关转回到应用系统。实现用户使用本校校园网身份，访问支持OAuth Client应用系统的目标。



## SP+OAuth应用系统访问流程

 ![CARSI](/CARSI_SP+OAuth网关方式接入SP.files/001.png)

1. 用户通过应用系统的服务页面提供的“通过CARSI登录”按钮，跳转到CARSI SP+OAuth网关的跳转链接（OAuth Server的authorize接口）。
2. OAuth Server通过Shibboleth SP将用户浏览器自动跳转到CARSI联盟的服务发现页面（https://ds.carsi.edu.cn）。用户在此页面选择所在学校，在本校IdP输入身份、完成认证。通过Shibboleth机制，自动回跳到SP+OAuth网关的指定页面。
3. 该指定页面自动进行二次回跳，跳转到应用系统的callback地址，也就是应用系统资源页面，附带认证成功的code参数。
4. 应用系统的资源页面（callback页面）通过code向网关上的OAuth Server获取token，OAuth认证过程结束。
5. 应用系统的资源页面（callback页面）通过token向网关上的OAuth Server获取资源信息，如用户身份属性。注意：用户身份信息为加密传输。

## 应用系统接入步骤

1. 首先完成如下准备工作：

   a)   在应用系统web服务页面，添加“通过CARSI登录”按钮。

   b)   配置OAuth callback函数，编写callback页面，显示认证成功后资源。

   c)   本地配置X.509证书和公私钥加密机制。公钥用于加密应用系统从网关获取的用户信息。

   

2. 发送邮件给[carsi@pku.edu.cn](mailto:carsi@pku.edu.cn)，提交以SP+OAuth方式接入应用系统的申请。邮件标题为：【XXX单位名称】申请以SP+OAuth方式接入应用系统。邮件正文包含以下内容：

   a)   应用系统URL及英文简称：CARSI联盟参考此信息生成应用系统的OAuth用户名，分配OAuth的账号（$client_id）、密码（$client_password）；

   b)   认证成功的callback地址（$callback_url）；

   c)   作为附件提交2048位的rsa公钥文件。

   

3. 配置应用系统支持OAuth服务

   a)   在应用系统登录页面添加“通过CARSI登录”按钮，CARSI OAuth认证的跳转链接：

```
href='https://spoauth.carsi.edu.cn/oauth2server/authorize.php?response_type=code&client_id=【填入申请的SP接入OAuth的账号（$client_id）】&state=carisi-sp'
```

​	b)   配置【认证成功的callback地址（$callback_url）】页面，支持get方式获取code参数，OAuth认证完成后将code参数自动传给【认证成功的callback地址（$callback_url）】；

​	c)   配置【认证成功的callback地址（$callback_url）】页面在获取到code之后，调用OAuth服务token接口，获得token和refresh token。

​	请使用以下服务测试网址，根据code获取token和refresh token。

```
{curl -u $client_id:$client_password https:// spoauth.carsi.edu.cn/oauth2server/token.php -d 'grant_type=authorization_code &code=【填入callback页面得到的code】'}， 
```

​	d)   配置【认证成功的callback地址（$callback_url）】页面在获取到token之后，调用OAuth服务的resource接口，获得对应的资源信息（即用户身份信息，如affiliation）。注意该信息使用添加服务时提供的公钥加密，需要使用对应的私钥解密。 

​	为便于测试，可以使用以下命令获取CARSI认证释放的用户身份信息（如affiliation信息）。

```
{curl https://spoauth.carsi.edu.cn/oauth2server/resource.php -d 'access_token =【填入获取到的access token】'} 
```

​	以下是【认证成功callback地址（$callback_url）】页面的php样例代码，供参考：

```
<?
session_start();
// store session data
$_SESSION['client_id']="test1.edu.cn";
$_SESSION['client_password']="xxxxxxxx";
$_SESSION['type']="authorization_code";
$server = "spoauth.carsi.edu.cn";

$code = null;
$callback  = null;
$access_token = null;
$refresh_token = null;
$expires = null;

if(isset($_GET["error"])) {
     exit($_GET["error"]."<br>".$_GET["error_description"]);
}

// store code in session and display it
if(isset($_REQUEST['code']))    $_SESSION['code'] = $_REQUEST['code'];
if(isset($_SESSION['code'])) echo "<h1>code:</h1>".$_SESSION['code'];
 
// display a token form
if(!isset($_SESSION['access_token'])){
     if (empty($_POST) or $_POST['token'] !== 'yes' ) {
         exit('

<form method="post">
  <label>token?</label><br />
  <input type="submit" name="token" value="yes">
  <input type="submit" name="token" value="no">
</form>');
     }

     // use code to get token from OAuth server
     $exec_lastline =exec("curl -u ".$_SESSION['client_id'].":".$_SESSION['client_password']."  https://$server/oauth2server/token.php -d 'grant_type=".$_SESSION['type']."&code=".$_SESSION['code']."'");
     $re = json_decode($exec_lastline);
     if(isset($re->access_token) ){
         $_SESSION['access_token']=$re->access_token;
         $_SESSION['refresh_token']= $re->refresh_token;
         $_SESSION['expires'] = $re->expires_in;
     }elseif (isset($re->error)){
         exit($re->error_description."<hr>");
     }
}

if(isset($_SESSION['access_token']))echo "<hr><h1>token:</h1>".$_SESSION['access_token'];
if(isset($_SESSION['refresh_token']))echo "<hr><h1>refresh:</h1>".$_SESSION['refresh_token'];
if(isset($_SESSION['expires']))echo "<hr><h1>expires:</h1>".$_SESSION['expires'];

// use token to get resource from OAuth server
if(true){
     $_SESSION['affiliation'] =base64_decode(exec("curl https://$server/oauth2server/resource.php -d 'access_token=".$_SESSION['access_token']."'"));
}
if(isset($_SESSION['affiliation'])) echo  "<hr><h1>affiliation(encrypted):</h1>".$_SESSION['affiliation'];

// decrypt affiliation with your own private key
if(isset($_SESSION['affiliation'])) {
     $privkey = "test1.edu.cn.key";
     $privkey = fread(fopen($privkey, "r"), filesize($privkey));
     openssl_private_decrypt($_SESSION['affiliation'], $decrypted, $privkey);
     $_SESSION['decrypted']= $decrypted;
}
if(isset($_SESSION['decrypted']))    echo  "<hr><h1>affiliation(decrypted):</h1>".$_SESSION['decrypted'];
```

4. 应用系统接入情况测试

   a)   访问调试完成的应用系统页面，点击“通过CARSI登录”按钮；

   b)   浏览器跳转到CARSI目录服务页面https://ds.carsi.edu.cn;

   c)   选择所在学校IdP进行身份认证；

   d)   认证成功后，浏览器被自动重定向到【callback页面】，并可现实相关resource信息，应用系统调试成功。

