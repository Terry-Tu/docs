# 1 前言

## 1.1 目的

本文档是云闪付开放平台对接入方的接口规范，本规范定义了接口，以及数据交换格式。

## 1.2 使用对象

本文档由前言、接口规范、接入流程、OAUTH2用户联登、后台接口对接、前台插件UPSDK、附录七部分组成，适用于合作双方技术人员、业务人员阅读。

# 2 接口规范

## 2.1 术语表

| 术语       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 数字签名   | 数据在网络中传输的安全手段之一，用于防篡改、验证数据收发双方身份，具体参见2.4安全规范部分 |
| 报文       | 数据传输的载体，由报文头、报文体、数字签名组成               |
| 参数       | 用于描述传输的数据信息，是报文、文件传输的基础组成部分       |
| 报文组件   | 由一个或多个参数组成，是一个可共用、复用的数据结构           |
| 请求发送方 | 报文或文件发送端                                             |
| 请求接收方 | 报文或文件接收端                                             |

 

## 2.2 报文规范 - API

报文在传输时采用UTF-8编码，采用json报文格式，返回报文由统一由"resp"、"params"、"msg"组成。

## 2.3 报文结构

请求报文结构



```java
//报文根节点
{
    //请求报文
    "xxx":"//报文参数",
    "xxx":"//报文参数"
}

```

应答报文结构

```java
//报文根节点
{
    "resp":"00",     //返回码  
    "msg":"",       //返回信息
    "params":{     //报文参数
        "xxx":"//报文参数",
        "xxx":"//报文参数"
    }
}
```

 

## 2.4 安全规范

### 2.4.1 报文签名(SHA256)

将所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。 Key，value均采用原始值，大小写不变，不进行URL 转义。最后对拼接字符串string1作sha256运算出signature。参见[附录二](#_附录二：SHA256签名)。

### 2.4.2 对称密钥解密 

symmetricKey 是银联分配给接入方的对称密钥，用于相关敏感信息的解密。加解密的方式为3DES，以base64编码输出，密钥以16进制的字符串形式提供。

解密方法参考如下：

```java
//用于将返回字段的值进行3DES解密
public static String getDecryptedValue(String value, String key) throws Exception {
		if (null == value || "".equals(value)) {
			return "";
		}
		byte[] valueByte = Base64.decode(value);
		byte[] sl = decrypt3DES(valueByte, BytesUtil.hexToBytes(key));
		String result = new String(sl);
		return result;
}

public static byte[] decrypt3DES(byte[] input, byte[] key) throws Exception {
		Cipher c = Cipher.getInstance("DESede/ECB/PKCS5Padding");
		c.init(Cipher.DECRYPT_MODE, new SecretKeySpec(key, "DESede"));
		return c.doFinal(input);
}

```



# 3 接入流程

![1615446319755](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446319755.png)

## 3.1 对接信息

应用主管方将接入参数通过申请表以工单形式发给云闪付事业部，技术对接信息如下：

Ø 接入方：XXX生活服务信息科技有限公司 //接入方全称。

Ø 接入方简称：YYY  //接入方简称。

Ø 联登授权域名：oauth.xxx.com    //只允许该域名使用联登接口。

Ø UPSDK安全域名：mobile..xxx.com //只允许该域名使用UPSDK。

Ø 后台对接通知域名：notify.xxx.com //只允许该域名作为通知地址。

Ø 后台加解密公钥：使用openssl生成，base64形式输出 //根据接口要求，部分接口需接入方采用私钥partnerKey对上送信息进行RSA签名，云闪付开放平台采用对应的公钥验证签名。

 

注意：

Ø 访问页面仅支持https。

Ø UPSDK安全域名，支持多个，长度不超过1024。

## 3.2 参考反馈

云闪付审核通过以后，将如下信息以工单形式反馈给接入方：

Ø appId: 接入方唯一标识。

Ø secret: 接入方密钥 //用于基础令牌接口的签名。

Ø symmetricKey: 对称密钥（3DES，16进制格式） //用于后台敏感数据解密。

Ø upPublicKey（银联方）: 使用openssl生成，base64形式输出 //根据接口要求，部分接口云闪付开放平台会采用私钥对返回信息进行RSA签名， 接入方使用公钥upPublicKey验证签名（特定业务接入需要，常规接入不涉及此参数）。

 

# 4 OAUTH2用户联登

## 4.1 使用场景

接入方必须通过联登获取云闪付用户openid，以识别用户身份，为用户提供个性化服务。同时，如有需要，接入方也可以在获得用户授权后，获取用户相关个人信息。

接入方可以通过如下几种scope获取相关信息等：

| scope          | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| upapi_base     | 基础授权，云闪付不会弹出授权页面，仅可以获取用户openId       |
| upapi_mobile   | 用户手机号授权，云闪付会弹出授权页面，仅可获取用户手机号     |
| upapi_auth     | 用户实名授权（手机号[可选]+姓名+证件号），云闪付会弹出授权页面，获取用户实名信息，亦可以获取用户手机号，但仍需调用获取手机号接口，具体返回请以用户是否授权获取为准 |
| upapi_contract | 无感支付 【授权链接参照如下】  https://open.95516.com/s/open/noPwd/html/open.html?  appId=XXX  &redirectUri=https://XXX/XXX  &responseType=code  &scope=upapi_contract  &planId=9ca088374a8141ff95dca35a12d73240 |

 

注：云闪付只会向特定行业开放实名认证要素信息

## 4.2 用户联登流程

![1615446394090](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446394090.png)

## 4.3 系统对接步骤

第一步：请求进入云闪付用户联登授权页，获取CODE

云闪付APP外的接入方APP，使用云闪付SDK相关接口直接获得code。调用方式参考SDK文档及示例代码。

 

云闪付APP内的接入方H5，跳转到此页面，请求获取code

https://open.95516.com/s/open/html/oauth.html?appId=APPID&redirectUri=REDIRECTURI&responseType=code&scope=SCOPE&state=STATE

 

云闪付APP外（如在微信公众号里）的接入方H5，跳转到此页面，用户登录后获得code

https://open.95516.com/s/open/auth/outApp/html/outLogin.html?appId=APPID&redirectUri=REDIRECTURI&responseType=code&scope=SCOPE&state=STATE

 

云闪付APP外的SCOPE仅可使用upapi_login（同upapi_base）和upapi_user（同upapi_auth）。

 

强烈建议：跳转回调redirectUri使用https链接来确保授权code的安全性

请求参数说明：

| 序号 | 参数名       | 是否必填 | 备注                                                         |
| :--- | ------------ | -------- | ------------------------------------------------------------ |
| 1.   | appId        | 是       | 接入方的唯一标识                                             |
| 2.   | redirectUri  | 是       | 授权后重定向的回调地址，请使用urlencode对链接进行处理（ios无需进行urlencode）。 |
| 3.   | responseType | 是       | 返回类型，请填写code                                         |
| 4.   | scope        | 是       | 应用授权作用域，参见4.1使用场景                              |
| 5.   | state        | 否       | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节 |
| 6.   | planId       | 否       | scope 为 upapi_contract  时上送。接入方定义的签约协议模板id，用于在签约页面中展示签约内容 |

 

登录之后页面将跳转至 redirectUri?code=CODE&state=STATE（成功获取code时）

或 redirectUri?state=STATE&errmsg=XXXXYYYY&errorcode=xxx （失败时）

注意： 

Ø 返回code是通过encodeURIComponent经过URL编码的，接入方需要通过函数decodeURIComponent解码去获取code。

 

第二步：调用云闪付开放平台接口，通过CODE获取accessToken和openId

通过步骤一获取的code，以及后台接口获取的backendToken，发送如下后台接口获取accessToken和openId

请求地址：

https://open.95516.com/open/access/1.0/token

 

输入参数说明：

| **序号** | **参数名**   | **是否必填** | **备注**                                                    |
| -------- | ------------ | ------------ | ----------------------------------------------------------- |
| 1.       | appId        | 是           | 接入方的唯一标识                                            |
| 2.       | backendToken | 是           | 通过后台接口“获取backendToken<backendToken>”获取，参见6.1.1 |
| 3.       | code         | 是           | 填写步骤一获取的code                                        |
| 4.       | grantType    | 是           | 常量字符串authorization_code                                |

 

 

返回参数说明：

| **序号** | **参数名**   | **备注**                                    |
| -------- | ------------ | ------------------------------------------- |
| 1.       | accessToken  | 网页授权接口调用凭证                        |
| 2.       | expiresIn    | accessToken接口调用凭证超时时间，单位（秒） |
| 3.       | refreshToken | 用户刷新accessToken（预留）                 |
| 4.       | openId       | 用户唯一标识                                |
| 5.       | scope        | 用户授权的作用域                            |

 

注意：

Ø accessToken有效期expiresIn为后台接口返回控制，现阶段设置为3600 秒。当accessToken超时后，需要用户重新授权。

Ø 如果接入方权限为基础授权(upapi_base)，在获取openId后，后续步骤则可不再执行。接入方权限为基础授权以上权限级别，可通过openId， 再去获取授权的其他资源信息。

Ø 如果仅仅做用户联登的话，无需关心accessToken有效期，开发者拿到accessToken以后，可直接执行第五步获取用户信息。

Ø 基于安全以及用户信息一致性考虑， 建议接入方不要将用户登录信息永久的记录在本地，可采用定时刷新的方式同步用户信息。

 

报文示例：

```java
请求报文：
{
	"appId":"fb707efe************4c3a9e06d85",
	"backendToken":"ip76w9FgSjaxuN0tajG4Rg=="
	"code":"KFTrLiCknhMlKKuv",
	"grantType":"authorization_code"
}
响应报文：
{
		"accessToken":"APwY8AIr7URGk2FP6zyO4IpoBCQ48+2iy3+b9TWfGmpTjJfbUlFKg8f5qJxUcxJd0POQFrP43cSFwdl1j7bUvxwvrp2/0iLs ",
	"expiresIn":"3600"
	"refreshToken":"KFTrLiCknhMlKKuv",
	"openId":"JUyCjGbfhTxlSKtGka9hAgTAc1hMOY40gCD6fcB/2I5OOEkEXYrlsx2oAalWQitX ",
	"scope":" upapi_base "
}


```

 

# 5 后台接口对接

## 5.1 基础接口

### 5.1.1 获取backendToken<backendToken>

#### 5.1.1.1功能描述

基础服务令牌，通过appId、secret换取，有效期为7200秒，后台接口调用凭证。

#### 5.1.1.2请求地址

https://open.95516.com/open/access/1.0/backendToken

#### 5.1.1.3报文结构

请求报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                                     |
| -------- | ---------- | ------------ | ------------------------------------------------------------ |
| 1.       | appId      | 是           | 接入方的唯一标识                                             |
| 2.       | nonceStr   | 是           | 生成签名的随机串                                             |
| 3.       | timestamp  | 是           | 生成签名的时间戳                                             |
| 4.       | signature  | 是           | 签名值，签名方法参见2.4.1报文签名  签名因子：appId、secret、nonceStr、timestamp |

 

响应报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1.       | backendToken |              | 后台接口调用凭证                             |
| 2.       | expiresIn    |              | backendToken接口调用凭证超时时间，单位（秒） |

 

注意：

Ø secret由云闪付分配给接入方的密钥，请求时无需上送，但需要作为签名因子进行签名。

Ø nonceStr生成规则参见附录三-生成签名随机字符串nonceStr。

Ø timestamp生成签名时间戳：接入方生成从1970年1月1日00:00:00至今的秒数，即当前的时间，单位为秒。

#### 5.1.1.4报文示例

```java
请求报文：
{
	"signature":"0819575ddf07ab5c8e25ac66dbf8b861764d05a2be59abe6bd52a9b7d4ca6eaa",
	"appId":"fb707efe************4c3a9e06d85",
	"nonceStr":"KFTrLiCknhMlKKuv",
	"timestamp":"1553425553"
}
响应报文：
{
    "msg":"",
    "params":{"backendToken":"ip76w9FgSjaxuN0tajG4Rg==","expiresIn":7200},
    "resp":"00"
}

```

### 5.1.2 获取frontToken<frontToken>

#### 5.1.2.1功能描述

基础服务令牌，通过appId、secret换取，有效期为7200秒，upsdk初始化凭证。

#### 5.1.2.2请求地址

[https://open.95516.com/open/access/1.0/](https://open.95516.com/open/access/1.0/backendToken)frontToken

#### 5.1.2.3报文结构

请求报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                                     |
| -------- | ---------- | ------------ | ------------------------------------------------------------ |
| 1.       | appId      | 是           | 接入方的唯一标识                                             |
| 2.       | nonceStr   | 是           | 生成签名的随机串                                             |
| 3.       | timestamp  | 是           | 生成签名的时间戳                                             |
| 4.       | signature  | 是           | 签名值，签名方法参见2.4.1报文签名  签名因子：appId、  secret、nonceStr、timestamp |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                   |
| -------- | ---------- | ------------ | ------------------------------------------ |
| 1.       | frontToken |              | UPSDK初始化凭证                            |
| 2.       | expiresIn  |              | frontToken接口调用凭证超时时间，单位（秒） |

 

​    注意：

Ø secret由云闪付分配给接入方的密钥，请求时无需上送，但需要作为签名因子进行签名。

Ø nonceStr生成规则参见附录三-生成签名随机字符串nonceStr。

Ø timestamp生成签名时间戳：接入方生成从1970年1月1日00:00:00至今的秒数，即当前的时间，单位为秒。

#### 5.1.2.4报文示例

```java
请求报文：
{
	"signature":"0819575ddf07ab5c8e25ac66dbf8b861764d05a2be59abe6bd52a9b7d4ca6eaa",
	"appId":"fb707efe************4c3a9e06d85",
	"nonceStr":"KFTrLiCknhMlKKuv",
	"timestamp":"1553425553"
}
响应报文：
{
    "msg":"",
    "params":{"backendToken":"ip76w9FgSjaxuN0tajG4Rg==","expiresIn":7200},
    "resp":"00"
}

```

 

## 5.2 获取用户信息

### 5.2.1 获取用户敏感信息-登陆手机号<user.mobile>

#### 5.2.1.1功能描述

用户联登且点击”确认授权”后，通过该接口可获取用户手机号码。

#### 5.2.1.2请求地址

https://open.95516.com/open/access/1.0/user.mobile

#### 5.2.1.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1.       | appId        | 是           | 接入方的唯一标识                             |
| 2.       | accessToken  | 是           | 全局唯一接口调用凭证，通过用户联登获取       |
| 3.       | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取         |
| 4.       | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**   |
| -------- | ---------- | ------------ | ---------- |
| 1.       | mobile     |              | 登录手机号 |

 

​    注意：

Ø 授权scope为upapi_mobile、upapi_auth、upapi_user的接入方才可以访问此接口。

Ø 此接口所有字段加密返回，请用云闪付分配给接入方的对称秘钥（symmetricKey）解密，加密内容为base64格式输出，参见安全规范2.4.2 对称密钥解密。

#### 5.2.1.4报文示例

```java
请求报文：
{
    "signature":"2c00a7100b97fc38a30b3e26c5e0c221cd21257c2804e48e1e799975a7884436",
    "appId":" fb707efe************4c3a9e06d85",
    "nonceStr":"vhLegXQJ0EMFkDJJ",
    "timestamp":"1553425554"
}
响应报文：
{
    "msg":"",
    "params":{
        "expiresIn":7200,
        "frontToken":"AqtwoCiBSKaJUDuOu5+mdA=="
    },
    "resp":"00"
}

```

### 5.2.2 获取用户信息-获取用户敏感信息-姓名+证件号码<user.auth>

#### 5.2.2.1功能描述

用户联登且点击”确认授权”后，后台通过该接口可获取用户证件信息。

#### 5.2.2.2请求地址

https://open.95516.com/open/access/1.0/user.auth

#### 5.2.2.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1.       | appId        | 是           | 接入方的唯一标识                             |
| 2.       | accessToken  | 是           | 全局唯一接口调用凭证，通过用户联登获取       |
| 3.       | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取         |
| 4.       | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                             |
| -------- | ---------- | ------------ | ---------------------------------------------------- |
| 1.       | realName   |              | 用户姓名                                             |
| 2.       | certTp     |              | 证件类型（01:身份证，03:护照，04:回乡证，05:台胞证） |
| 3.       | certId     |              | 证件号                                               |

 

​    注意：

Ø 授权scope为upapi_auth、upapi_user的特定接入方才能访问此接口。

Ø 此接口所有字段加密返回，请用云闪付分配给接入方的对称秘钥（symmetricKey）解密，加密内容为base64格式输出，参见安全规范2.4.2 对称密钥解密。

 

#### 5.2.2.4报文示例

```java
请求报文：
{
    "backendToken":"LMvASreQS9eazNFQ216zZw==",
    "openId":"JUyCjGbfhTxlSKtGka9hAgTAc1hMOY40gCD6fcB/2I5OOEkEXYrlsx2oAalWQitX",
    "appId":"fb707efe************4c3a9e06d85",
    "accessToken":"APwY8AIr7URGk2FP6zyO4IpoBCQ48+2iy3+b9TWfGmpTjJfbUlFKg8f5qJxUcxJd0POQFrP43cSFwdl1j7bUvxwvrp2/0iLs"
}
响应报文：
{
    "msg":"",
    "params":{
    		"certId":"EhPC5IKLZ+HfLs/ect9K7mi4q6RUcmsj",
            "certTp":"IJabTPvcSM4=",
            "realName":"RGfb8ZmFGCI="
        },
    "resp":"00"
}

```

### 5.2.3 用户授权解除通知接口

#### 5.2.3.1功能描述

调用第三方回调地址，通知信息授权解除结果。云闪付APP-安全中心-账号授权入口下进行解除信息授权使用。

#### 5.2.3.2请求地址

第三方回调地址。

#### 5.2.3.3报文结构

请求报文参数说明

| **序号** | **参数名** | **备注**                                    |
| -------- | ---------- | ------------------------------------------- |
| 1        | appId      | 接入方的唯一标识                            |
| 2        | timestamp  | 生成签名的时间戳                            |
| 3        | nonceStr   | 生成签名的随机串                            |
| 4        | openId     | OAUTH2中获取的openId                        |
| 5        | signature  | 请使用银联的公钥验证签名，输出格式为base64. |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                  |
| -------- | ---------- | ------------ | ----------------------------------------- |
| 1.       | resp       |              | 成功接收通知以后发送  {"resp": "00"} 返回 |

 

​    注意：

Ø 通知地址是入网申请时提交的后台对接通知域名。

Ø 示例代码可参考5.4.3通知解约结果接口。

 

## 5.3 支付密码验证

### 5.3.1 支付密码验证-获取支付密码验证流水号<verifyPwdSeqId>

#### 5.3.1.1功能描述

云闪付支付密码验证，通过”支付交易信息”换取”支付密码验证流水号”，供 upsdk（支付密码授权）接口使用。

#### 5.3.1.2请求地址

https://open.95516.com/open/access/1.0/verifyPwdSeqId

#### 5.3.1.3报文结构

请求报文参数说明

| **序号** | **参数名**  | **是否必填** | **备注**                                             |
| -------- | ----------- | ------------ | ---------------------------------------------------- |
| 1.       | appId       | 是           | 接入方的唯一标识                                     |
| 2.       | openId      | 是           | 第三方用户唯一凭证，通过用户联登获取                 |
| 3.       | clientIp    | 否           | 终端IP                                               |
| 4.       | bizOrderId  | 是           | 业务操作流水号                                       |
| 5.       | bizType     | 是           | 业务类型 （购买基金:4a）                             |
| 6.       | merchantId  | 是           | 商户号                                               |
| 7.       | nonceStr    | 是           | 生成签名的随机串                                     |
| 8.       | notifyUrl   | 是           | 通知地址                                             |
| 9.       | timestamp   | 是           | 生成签名的时间戳                                     |
| 10.      | accessToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取         |
| 11.      | signature   | 是           | 签名值，签名方法参见2.4.1报文签名  签名因子:1-10字段 |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**             |
| -------- | ---------- | ------------ | -------------------- |
| 1.       | transNo    |              | 订单号，业务唯一标识 |

 

​    注意：

Ø clientIp，终端IP服务端IP地址。

Ø bizOrderId，业务操作流水号，由接入方生成需保证唯一，用于标识一笔唯一的支付密码验证交易。

Ø bizType，业务类型，请与云闪付业务方确定。

Ø notifyUrl，通知地址，请填写用于接收支付密码验证结果的服务地址。

#### 5.3.1.4报文示例

```java
请求报文：
{
    "bizType":"4h",
    "bizOrderId":"711076926887122",
    "merchantId":"474919534200503",
  "signature":"d8A2wRrCrm4S9Eak7M4SqHXj2WNOlRfuad7xF86cgQdzdiKGXX71pOcaEoErWxdFrJ9WPQ7PVmpH5DfOzrX88ej8DAf4uH5yjhGWc3dwgZ69yoHjH6Wl6esMB7/QEPB/JuEkMS4lRmUsDg3R6Uj4XzLAjbfdg8awiKAuaUx1/Yo=",
    "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
    "appId":"fb707efe************4c3a9e06d85",
    "clientIp":"192.168.8.104",
    "notifyUrl":"https://XXX7.com",
  "accessToken":"APwY8AIr7UTGLSH1jn+psJUlPU9Ew8osvA7wuCoq2nPFg1qkM1lnXiwEwR+SlYCjTkc//JjyyQNAS8KT6l6eCGsF2SOA6Z3oBf0dV/9avfw=",
    "nonceStr":"JoErQGVrVy7MT70t",
    "timestamp":"1553426982"
}
响应报文：
{
    "msg":"",
    "params":{
            "transNo":"216609E51C1649DD94231B720FDE1DD9"
        },
    "resp":"00"
}

```



### 5.3.2 通知支付密码验证结果

#### 5.3.2.1功能描述

云闪付支付密码验证，根据”获取支付密码验证流水号”接口设置的通知地址（notifyUrl），云闪付将用户密码验证的结果通知接入方。

#### 5.3.2.2报文结构

通知报文参数说明

| **序号** | **参数名** | **备注**                                               |
| -------- | ---------- | ------------------------------------------------------ |
| 1.       | appId      | 接入方的唯一标识                                       |
| 2.       | timestamp  | 生成签名的时间戳                                       |
| 3.       | nonceStr   | 生成签名的随机串                                       |
| 4.       | openId     | OAUTH2中获取的openId                                   |
| 5.       | bizOrderId | 业务操作流水号                                         |
| 6.       | merchantId | 商户号                                                 |
| 7.       | bizType    | 业务类型                                               |
| 8.       | transNo    | 订单号，业务唯一标识                                   |
| 9.       | signature  | 输出格式为base64，请使用银联的公钥验证签名，参见附录五 |

 

返回报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                  |
| -------- | ---------- | ------------ | ----------------------------------------- |
| 1.       | resp       | 是           | 成功接收通知以后发送  {"resp": "00"} 返回 |

 

​    注意：

Ø 云闪付侧返回订单验证结果，使用云闪付私钥进行签名，接入方需要使用云闪付发布的公钥upPublicKey对收到的通知消息进行验证签名。

#### 5.3.2.3报文示例

```java
请求报文：
{
    "bizType":"4h",
    "bizOrderId":"711076926887122",
    "merchantId":"474919534200503",
    "signature":"d8A2wRrCrm4S9Eak7M4SqHXj2WNOlRfuad7xF86cgQdzdiKGXX71pOcaEoErWxdFrJ9WPQ7PVmpH5DfOzrX88ej8DAf4uH5yjhGWc3dwgZ69yoHjH6Wl6esMB7/QEPB/JuEkMS4lRmUsDg3R6Uj4XzLAjbfdg8awiKAuaUx1/Yo=",
    "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
    "appId":"fb707efe************4c3a9e06d85",
    "nonceStr":"JoErQGVrVy7MT70t",
    " transNo":"216609E51C1649DD94231B720FDE1DD9",
    "timestamp":"1553426982"
}

响应报文：
{
    "resp":"00"
}

```

 

## 5.4 无感支付

### 5.4.1 申请签约<contract.apply>

#### 5.4.1.1功能描述

无感支付签约接口，用户联登(SCOPE:upapi_contract)且点击”同意授权”后，后台可通过此接口发起支付签约。业务流程参考附录九：用户无感支付签解约流程。

#### 5.4.1.2请求地址

https://open.95516.com/open/access/1.0/contract.apply

#### 5.4.1.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1.       | appId        | 是           | 接入方的唯一标识                             |
| 2.       | accessToken  | 是           | 全局唯一接口调用凭证，通过用户联登获取       |
| 3.       | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取         |
| 4.       | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |
| 5.       | planId       | 是           | 协议模板id，由云闪付录入模板并分配给接入方   |
| 6.       | contractCode | 是           | 接入方侧的签约协议号，由接入方自行生成       |

 

响应报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                               |
| -------- | ------------ | ------------ | ------------------------------------------------------ |
| 1.       | contractCode |              | 接入方传来的签约协议号                                 |
| 2.       | planId       |              | 接入方传来的协议模板id                                 |
| 3.       | openId       |              | 凭此获取云闪付APP中接入方的appid下用户的唯一标识openid |
| 4.       | operateTime  |              | 操作时间                                               |
| 5.       | contractId   |              | 签约成功后，云闪付返回的委托免密支付协议 id。          |

 

​    注意：

Ø 只有授权scope为upapi_contract的特定接入方才能访问此接口。

#### 5.4.1.4报文示例

```java
请求报文：
{
    "backendToken":"ChXumdILQ1ej4bNkgrotlA==",
    "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
    "appId":"fb707efe************4c3a9e06d85",
    "planId":"11495f99471*****bcdecdd2bee79491",
    "accessToken":"APwY8AIr7USrbA1PWZPu3W9CDn7BCDgPEezSAQxoleTm9/d3JKRf/OXC6Nx8Rot290gcCakHerS1YWc4OYe6F5Us58HxUGIcKeD56ptOUws=",
    "contractCode":"684916616616"
}
响应报文：
{
    "msg":"",
    "params":{
            "contractCode":"684916616616",
        "contractId":"6218400000002496717", "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
            "operationTime":"20190324203725",
            "planId":"11495f99471*****bcdecdd2bee79491"
        },
    "resp":"00"
}

```

### 5.4.2 申请解约<contract.relieve>

#### 5.4.2.1功能描述

无感支付解约，使用接入方签约时传入的”签约协议号”，通过此接口可发起无感支付解约。

#### 5.4.2.2请求地址

https://open.95516.com/open/access/1.0/contract.relieve

#### 5.4.2.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                               |
| -------- | ------------ | ------------ | ------------------------------------------------------ |
| 1.       | appId        | 是           | 接入方的唯一标识                                       |
| 2.       | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取                   |
| 3.       | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取           |
| 4.       | contractId   | 是           | 委托免密支付签约成功后由云闪付返回的委托免密支付协议id |
| 5.       | planId       | 是           | 协议模板id，由云闪付录入模板并分配给接入方             |
| 6.       | contractCode | 是           | 接入方请求签约时传入的签约协议号，由接入方生产且须唯一 |

 

响应报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                               |
| -------- | ------------ | ------------ | ------------------------------------------------------ |
| 1.       | contractCode |              | 接入方传来的签约协议号                                 |
| 2.       | planId       |              | 接入方传来的协议模板id                                 |
| 3.       | openId       |              | 凭此获取云闪付APP中接入方的appId下用户的唯一标识openId |
| 4.       | operateTime  |              | 操作时间                                               |

 

​    注意：

Ø 只有授权scope为upapi_contract的特定接入方才能访问此接口。

#### 5.4.2.4报文示例

```java
请求报文：
{
    "backendToken":"B4fdbIpdSzuirV1J+8UHvQ==",
    "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
    "appId":"fb707efe************4c3a9e06d85",
    "contractId":"6218400000002487633",
    "planId":"11495f99471*****bcdecdd2bee79491",
    "contractCode":"655558569874523"
}
响应报文：
{
    "msg":"",
    "params":{
    "contractCode":"655558569874523",
    "openId":"42q4vi0Ae+8oe0R0dx0SMSYF2HCjVHWSp5qN5DukLHxhc9uRqTOD4swVOlhc86M4",
            "operationTime":"2019-03-24 18:38:27",
            "planId":"11495f99471*****bcdecdd2bee79491"
        },
    "resp":"00"
}

```



 

### 5.4.3 通知解约结果

#### 5.4.3.1功能描述

调用第三方回调地址，通知解约结果。云闪付APP-设置-支付设置-无感支付入口下进行无感支付解约使用，公交地铁脱机场景不支持云闪付侧解约。

#### 5.4.3.2请求地址

第三方回调地址。入网时申请表填写的解约结果通知地址。

#### 5.4.3.3报文结构

请求报文参数说明

| **序号** | **参数名**  | **是否必填** | **备注**                                                     |
| -------- | ----------- | ------------ | ------------------------------------------------------------ |
| 1.       | appId       | 是           | 接入方的唯一标识                                             |
| 2.       | timestamp   | 是           | 生成签名的时间戳                                             |
| 3.       | nonceStr    | 是           | 生成签名的随机串                                             |
| 4.       | operateTime | 是           | 操作时间                                                     |
| 5.       | openId      | 是           | 第三方用户唯一凭证，通过用户联登获取                         |
| 6.       | trid        | 是           | 特定场景使用                                                 |
| 7.       | contractId  | 是           | 签约成功后，云闪付返回的签约协议号，接入方凭此发起扣款，返回数据格式为AN32 |
| 8.       | signature   | 是           | 输出格式为base64，请使用银联的公钥验证签名，参见附录五       |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                  |
| -------- | ---------- | ------------ | ----------------------------------------- |
| 1.       | resp       |              | 成功接收通知以后发送  {"resp": "00"} 返回 |

 

​    注意：

Ø 只有授权scope为upapi_contract的特定接入方才能访问此接口。

#### 5.4.3.4报文示例-成功

```java
请求报文：
GET /template/api/testTemp
			?timestamp=1568251930
			&appId=fb707ef*********ae4c3a9e06d85
			&trid=62009000052
			&operateTime=2019-11-05 10:45:55
			&nonceStr=XFg3Lx4fUTaFyFLJ
			&contractId=6218400000020001614
			&signature=FavJObjrtxIf8z+qtTj8Tmlyw/tyyihEZ8TE5VdCRd4MI0ZnOJVOlo4MhdANIZ+TC3cYLBIWDZzBiUOCe6FasvW3+tAV2aLJkMR3Q5QjJ6O2IcMWcCOc6rxUMFBj5PNmQpLhfDiZxoj9DFKgkFwJqIvTwOiN+4G5n1d2i4u/KDeK4RL/oH0arrFQIJeiGSq79ds1A6YBXRkp37mga9TQLAeJIJNwzkPgxUa+uF30BVfde6+2K2jvpBFm7j8tLfCHXsPjbqLyUJKFLvDGFBwEhwlKAGFur9y/2slZxDOhbP8M1fWuzQhjxEWyAkA6vFtOl8p7lengdx+MXZ0DWKUiQ==
			&openId=A+2pMHJdXtQ42kQsuqUyM4qPubW5hrr3ONwj7Dgnu2wCFGcrnYVZiUo5AbUZPnBm
响应报文：
{'resp': '00'}

```

#### 5.4.3.5报文示例-失败（验签不通过）

```java
请求报文：
GET /template/api/testTemp
			?timestamp=1568251930
			&appId=fb707ef*********ae4c3a9e06d85
			&trid=62009000052
			&operateTime=2019-11-05 10:45:55
			&nonceStr=XFg3Lx4fUTaFyFLJ
			&contractId=6218400000020001614
			&signature=FavJObjrtxIf8z+qtTj8Tmlyw/tyyihEZ8TE5VdCRd4MI0ZnOJVOlo4MhdANIZ+TC3cYLBIWDZzBiUOCe6FasvW3+tAV2aLJkMR3Q5QjJ6O2IcMWcCOc6rxUMFBj5PNmQpLhfDiZxoj9DFKgkFwJqIvTwOiN+4G5n1d2i4u/KDeK4RL/oH0arrFQIJeiGSq79ds1A6YBXRkp37mga9TQLAeJIJNwzkPgxUa+uF30BVfde6+2K2jvpBFm7j8tLfCHXsPjbqLyUJKFLvDGFBwEhwlKAGFur9y/2slZxDOhbP8M1fWuzQhjxEWyAkA6vFtOl8p7lengdx+MXZ0DWKUiQ==
			&openId=A+2pMHJdXtQ42kQsuqUyM4qPubW5hrr3ONwj7Dgnu2wCFGcrnYVZiUo5AbUZPnBm
响应报文：
{'resp': '01'}

```

#### 5.4.3.6报文示例-异常

```java
请求报文：
GET /template/api/testTemp
			?timestamp=1568251930
			&appId=fb707ef*********ae4c3a9e06d85
			&trid=62009000052
			&operateTime=2019-11-05 10:45:55
			&nonceStr=XFg3Lx4fUTaFyFLJ
			&contractId=6218400000020001614
	&signature=FavJObjrtxIf8z+qtTj8Tmlyw/tyyihEZ8TE5VdCRd4MI0ZnOJVOlo4MhdANIZ+TC3cYLBIWDZzBiUOCe6FasvW3+tAV2aLJkMR3Q5QjJ6O2IcMWcCOc6rxUMFBj5PNmQpLhfDiZxoj9DFKgkFwJqIvTwOiN+4G5n1d2i4u/KDeK4RL/oH0arrFQIJeiGSq79ds1A6YBXRkp37mga9TQLAeJIJNwzkPgxUa+uF30BVfde6+2K2jvpBFm7j8tLfCHXsPjbqLyUJKFLvDGFBwEhwlKAGFur9y/2slZxDOhbP8M1fWuzQhjxEWyAkA6vFtOl8p7lengdx+MXZ0DWKUiQ==		&openId=A+2pMHJdXtQ42kQsuqUyM4qPubW5hrr3ONwj7Dgnu2wCFGcrnYVZiUo5AbUZPnBm
响应报文：
{
    "errorMessage": "系统错误，请稍后重试",
    "errorCode": "35000"
}

```

 

### 5.4.4 用户状态查询<contract.status>

#### 5.4.4.1功能描述

查询用户在云闪付侧是否存在无感支付业务未完成订单。

#### 5.4.4.2请求地址

https://open.95516.com/open/access/1.0/contract.status

#### 5.4.4.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1.       | appId        | 是           | 接入方的唯一标识                             |
| 2.       | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取         |
| 3.       | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                                  |
| -------- | ---------- | ------------ | --------------------------------------------------------- |
| 1.       | enable     |              | 返回0，表示无未完成订单；返回1-N，表示存在N笔未完成订单。 |

 

注意：

Ø 只有授权scope为upapi_contract的特定接入方才能访问此接口。

#### 5.4.4.4报文示例

```java
请求报文：
{
    "backendToken":"ovM8WAeKSG2C/A/lco86ew==",
    "openId":"JUyCjGbfhTxlSKtGka9hAgTAc1hMOY40gCD6fcB/2I5OOEkEXYrlsx2oAalWQitX",
    "appId":" fb707efe************4c3a9e06d85"
}
响应报文：
{
    "msg":"",
    "params":{
    "enable":"0"
},
    "resp":"00"
}

```



### 5.4.5 签约关系查询<contract.info>

#### 5.4.5.1 功能描述

查询接入方用户在云闪付的签约关系。

#### 5.4.5.2 请求地址

[https://open.95516.com/open/access/1.0/contract. info ](https://open.95516.com/open/access/1.0/contract.status)

#### 5.4.5.3 报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1        | appId        | 是           | 接入方的唯一标识                             |
| 2        | openId       | 是           | 第三方用户唯一凭证，通过用户联登获取         |
| 3        | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |
| 4.       | planId       | 是           | 协议模板id，由云闪付录入模板并分配给接入方   |

 

响应报文参数说明

| **序号** | **参数名**     | **是否必填** | **备注**                               |
| -------- | -------------- | ------------ | -------------------------------------- |
| 1.       | contractId     |              | 签约协议号                             |
| 2        | contractStatus |              | 签约状态：空-未签约 1-已签约  3-已解约 |

#### 5.4.5.4 报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "backendToken": "i6i9******************A==",
		"openId": "AJiPZzXPcYA64Kule7zhCUiR+QgWxAb0pDwho+gy6T9sf7w1tWMq/H+4Prle9KuN",
   "planId":"11495f99471f4c1cbcdecdd2bee79491"
}
响应报文：
{
    "result": {
                {
            "resp": "00",
            "msg": "成功",
            "params": {
                "contractStatus": "3",
                "contractId": "6218400000002450474"
            }
        }
    }
}

```

 

## 5.5 消息推送

### 5.5.1 消息推送（一个月）<new.msg.push>

#### 5.5.1.1 功能描述

消息推送接口。

#### 5.5.1.2 请求地址

https://open.95516.com/open/access/1.0/new.msg.push 

#### 5.5.1.3 报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                              |
| -------- | ------------ | ------------ | ----------------------------------------------------- |
| 1        | appId        | 是           | 接入方的唯一标识                                      |
| 2        | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取          |
| 3        | openId       | 是           | 用户openId                                            |
| 4        | content      | 是           | 推送的内容（最大支持140个字符，若为中文，则大小为70） |
| 5        | url          | 是           | 接入方传递的url（必须是在提供的UPSDK安全域名内）      |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                  |
| -------- | ---------- | ------------ | ------------------------- |
| 1        | appId      |              | 接入方的唯一标识          |
| 2        | openId     |              | 用户openId                |
| 3        | resultCd   |              | 应答码：00-成功，其他失败 |

#### 5.5.1.4 报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "backendToken": "i6i9******************A==",
		"openId": "AJiPZzXPcYA64Kule7zhCUiR+QgWxAb0pDwho+gy6T9sf7w1tWMq/H+4Prle9KuN",
        "content": "推送的消息",
        "url": "https://opentools.95516.com/msgPush"
}
响应报文：
{
"result": {
        "msg": "成功",
        "params": {
           "appId": "99eb12************************a01ba",
          "openId": "AJiPZzXPcYA64Kule7zhCUiR+QgWxAb0pDwho+gy6T9sf7w1tWMq/H+4Prle9KuN",
            "resultCd": "00"
        },
        "resp": "00"
}

```



 

## 5.6 实时退税

### 5.6.1 实时退税<return.tax>

#### 5.6.1.1 功能描述

接入方用户在云闪付APP海外实时退税操作接口。

#### 5.6.1.2 请求地址

[https://open.95516.com/open/access/1.0/return.tax ](https://open.95516.com/open/access/1.0/contract.status)

#### 5.6.1.3 报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                     |
| -------- | ------------ | ------------ | -------------------------------------------- |
| 1        | appId        | 是           | 接入方的唯一标识                             |
| 2        | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取 |
| 3        | refundNo     | 是           | 退税号                                       |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                  |
| -------- | ---------- | ------------ | ----------------------------------------- |
| 1.       | cardNo     |              | 卡号（加密）                              |
| 2        | cardTyoe   |              | 卡类型 （1借记 2贷记 3准贷记 4 借贷合一） |
| 3        | resp       |              | 成功通知以后发送 {‘resp’: ‘00’} 返回      |

#### 5.6.1.4 报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "backendToken": "diB8******************Q==",
        "refundNo": "1232144356436"
}
响应报文：
{
"result": {
        "msg": "成功",
        "params": {
            "cardNo": "",
            "cardType": "",
            "resp": ""
        },
        "resp": "00"
}

```

 

## 5.7 行业卡

### 5.7.1 行业卡操作<putIndustryCard >

#### 5.7.1.1 功能描述

接入方用户在云闪付APP编辑行业卡（增，删，改）操作。

#### 5.7.1.2 请求地址

https://open.95516.com/open/access/1.0/putIndustryCard 

#### 5.7.1.3 报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                                     |
| -------- | ------------ | ------------ | ------------------------------------------------------------ |
| 1        | appId        | 是           | 接入方的唯一标识                                             |
| 2        | backendToken | 是           | 基础服务令牌，通过”获取backendToken”接口获取                 |
| 3        | cardModuleId | 是           | cardModuleId行业卡 卡片模板id（需向云闪付提交申请）          |
| 4        | bussCardNo   | 是           | 行业卡 卡号                                                  |
| 5        | cardSubTp    | 是           | 行业卡 卡片子类型，   用于表示卡等级或者商户/机构下的不同额度的卡片 |
| 6        | transTp      | 是           | 交易类型 01：添加； 02：更新； 03：删除                      |
| 7        | openId       | 是           | OAUTH2中获取的openId                                         |
| 8        | region       | 否           | 签发地区代码（云闪付地区代码）                               |
| 9        | channelNo    | 否           | 社保金融服务平台编号                                         |
| 10       | name         | 否           | 姓名（若上传，需加密）                                       |
| 11       | certId       | 否           | 身份证号（若上传，需加密）                                   |
| 12       | mobileNo     | 否           | 手机号码（若上传，需加密）                                   |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                             |
| -------- | ---------- | ------------ | ------------------------------------ |
| 1        | resp       |              | 成功通知以后发送 {‘resp’: ‘00’} 返回 |

#### 5.7.1.4 报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "backendToken": "i6i9******************A==",
        "cardModuleId": "213454765435423",
        "bussCardNo": "12321325347654",
        "cardSubTp": "1",
        "transTp": "01",
        "openId": "AJiPZzXPcYA64Kule7zhCUiR+QgWxAb0pDwho+gy6T9sf7w1tWMq/H+4Prle9KuN",
        "region": "200120",
        "channelNo": "12343534654756",
        "name": "YngC5ECEDuQ=",
        "certId": "FzBGmOPyOwD4GyztXlqx74J5DKFX8NRB",
        "mobileNo": "YRpIAfdvpf17mlI5jdxr7A=="
}
响应报文：
{
"msg": "",
        "params": {
            "resp": ""
        },
        "resp": "00"
}

```

#### 说明：加密字段，请用云闪付分配给接入方的对称秘钥（symmetricKey）加密，加密内容为base64格式输出。加密方式：3DES，填充算法：DESede/ECB/PKCS5Padding。

## 5.8 红包相关

红包相关接入流程详见文末《附录十：红包/抽奖接入流程指引》。

### 5.8.1 红包赠送<red.packet.give>

#### 5.8.1.1 功能描述

接入方在云闪付进行红包赠送操作接口。

#### 5.8.1.2 请求地址

https://open.95516.com/open/access/1.0/red.packet.give

#### 5.8.1.3 报文结构

请求报文参数说明

| **序号** | **参数名**  | **是否必填** | **备注**                                                     |
| -------- | ----------- | ------------ | ------------------------------------------------------------ |
| 1        | appId       | 是           | 接入方的唯一标识                                             |
| 2        | insAcctId   | 是           | 机构账户代码 必须18位                                        |
| 3        | transNumber | 是           | 接入方交易流水号                                             |
| 4        | openId      | 否           | 用户标识                                                     |
| 5        | mobile      | 否           | 手机号使用symmetricKey 对称加密，内容为base64格式            |
| 6        | pan         | 否           | 同名卡号使用symmetricKey 对称加密，内容为base64格式          |
| 7        | pointAt     | 是           | 交易金额   单位：分，不能为负  （以分为单位，使用symmetricKey 对称加密，内容为base64格式） |
| 8        | remark      | 否           | 备注信息                                                     |
| 9        | nonceStr    | 是           | 生成签名的随机串                                             |
| 10       | timestamp   | 是           | 生成签名的时间戳                                             |
| 11       | signature   | 是           | 请使用接入方私钥签名，输出格式为base64.                      |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                             |
| -------- | ---------- | ------------ | ------------------------------------ |
| 1        | resp       |              | 成功通知以后发送 {‘resp’: ‘00’} 返回 |

说明：

1.接入方使用私钥进行签名，云闪付使用接入方提供的的公钥进行验证签名。

2.openId、mobile、pan 3个参数不能同时为空，至少上送一个。

\3. mobile、pan、pointAt字段加密（使用云闪付提供symmetricKey 对称加密，内容为base64格式）上送，但参与签名时，请使用明文加签。

\4. 除了signature之外，请求参数中所有字段参与验签。所有待签名参数（字符编码为 UTF-8）按照字段名的ASCII 码从小到大排序（字典序）后，使用键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。

对拼接字符串string1进行RSA签名后，按base64格式出现在报文中。

#### 5.8.1.4 报文示例

```java
请求报文：
{
 "appId": "99eb12************************a01ba",
        "insAcctId": "T00T00000000888527",
        "transNumber": "20191029163610128532046164531",
        "openId": "AJiPZzXPcYA64Kule7zhCUiR+QgWxAb0pDwho+gy6T9sf7w1tWMq/H+4Prle9KuN",
        "mobile": "YRpIAfdvpf17mlI5jdxr7A==",
        "pan": "",
        "pointAt": "tPAcG+FFZu4=",
        "remark": "",
        "nonceStr": "ZSVPXQw8I1HYsnpL",
        "timestamp": "1572338170",
        "signature": "YXBDgrYeywL3jfyzYbOW9wWLv1X69OYiu65x207Gw+a0TcSPzeN+oIuxQSZgSRqmUihRMJ9i3S7Q+jgeUP2Ko3A2qM7RBPkrblcUdALgsGNSqnT4g21uiErlPfvOdI3bTu5IfGhF1yBwFBDZ+X5aRprpKXkfJTBUPQBRaaJJqRk="}
响应报文：
{
  "msg": "成功",
        "params": {
            "resp": ""
        },
        "resp": "00"
}
```



### 5.8.2 红包 (机构账户)余额查询<red.packet.select>

#### 5.8.2.16.8.2.1功能描述

实时查询接入方红包(机构账户)余额数据接口。

#### 5.8.2.26.8.2.2请求地址

https://open.95516.com/open/access/1.0/red.packet.select

#### 5.8.2.36.8.2.3报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                    |
| -------- | ------------ | ------------ | ------------------------------------------- |
| 1        | appId        | 是           | 接入方的唯一标识                            |
| 2        | insAcctId    | 是           | 机构账户代码 必须18位                       |
| 3        | transNumber  | 是           | 接入方交易流水号                            |
| 4        | transDt      | 是           | 请求日期 YYYYMMDD例：20190712               |
| 5        | transTm      | 是           | 请求时间 YYYYMMDDHHMMSS  例：20190712144820 |
| 6        | backendToken | 是           | 后台接口授权token                           |

 

响应报文参数说明

| **序号** | **参数名**     | **是否必填** | **备注**                                                  |
| -------- | -------------- | ------------ | --------------------------------------------------------- |
| 1        | acctSt         |              | 账户状态  （使用symmetricKey 对称加密，内容为base64格式） |
| 2        | acctBalance    |              | 账户余额  （使用symmetricKey 对称加密，内容为base64格式） |
| 3        | validBeginDtTm |              | 有效开始时间                                              |
| 4        | validEndDtTm   |              | 有效结束时间                                              |

#### 5.8.2.1.4报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "insAcctId": "T00T00000000888527",
        "transNumber": "1320190910000368",
        "transDt": "20191021",
        "transTm": "20191021142022",
        "backendToken": "i6i9******************A=="
}
响应报文：
{
 "msg": "成功",
        "params": {
            "acctSt": "g1fkOyACkbI=",
            "acctBalance": "YSPlWlpz2Wc=",
            "validBeginDtTm": "",
            "validEndDtTm": ""
        },
   "resp": "00"
}

```



### 5.8.3 抽奖资格查询<qual.select>

#### 5.8.3.1 功能描述

接入方在云闪付查询用户抽奖资格接口。

#### 5.8.3.2 请求地址

https://open.95516.com/open/access/1.0/qual.select

#### 5.8.3.3 报文结构

请求报文参数说明

| **序号** | **参数名**   | **是否必填** | **备注**                                                |
| -------- | ------------ | ------------ | ------------------------------------------------------- |
| 1        | appId        | 是           | 接入方的唯一标识                                        |
| 2        | qualNum      | 是           | 资格池编号                                              |
| 3        | qualType     | 是           | 资格类型  固定值“open_id”、“mobile”、“card_no”          |
| 4        | qualValue    | 是           | 资格值  （使用symmetricKey 对称加密，内容为base64格式） |
| 5        | backendToken | 是           | 后台接口授权token                                       |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                                                     |
| -------- | ---------- | ------------ | ------------------------------------------------------------ |
| 1        | qualMap    |              | 满足要求的资格为map结构，key是资格有效期拼接格式如20171108_20171101（结束日期在前），value为资格次数 |

#### 5.8.3.4 报文示例

```java
请求报文：
{
	    "qualNum":"681ee844-40de-0376-1257-3ace7bc3a7c2",
		"qualType":"mobile",
		"qualValue":"1",
		"appId":"99eb12************************a01ba",
	    "backendToken":"i6i9******************A=="}
响应报文：
{
    "resp":"00",
    "msg":"成功",
    "params":{
    "qualMap":{},
    "respCode":"00",
    "respMsg":"成功"
	}
}

```

 

### 5.8.4 抽奖资格赠送<qual.send>

#### 5.8.4.1 功能描述

接入方给用户赠送抽奖资格接口。

#### 5.8.4.2 请求地址

https://open.95516.com/open/access/1.0/qual.send

#### 5.8.4.3 报文结构

请求报文参数说明

| **序号** | **参数名**  | **是否必填** | **备注**                                                     |
| -------- | ----------- | ------------ | ------------------------------------------------------------ |
| 1        | appId       | 是           | 接入方的唯一标识                                             |
| 2        | qualNum     | 是           | 资格池编号                                                   |
| 3        | qualType    | 是           | 资格类型  固定值“open_id”、“mobile”、“card_no”               |
| 4        | qualValue   | 是           | 资格值  （使用symmetricKey 对称加密，内容为base64格式）      |
| 5        | count       | 是           | 资格增加次数  （使用symmetricKey 对称加密，内容为base64格式） |
| 6        | transNumber | 是           | 流水号（确保唯一）                                           |
| 7        | startDate   | 是           | 资格起始时间 格式为yyyyMMdd                                  |
| 8        | endDate     | 是           | 资格结束时间 格式为yyyyMMdd                                  |
| 9        | nonceStr    | 是           | 生成签名的随机串                                             |
| 10       | timestamp   | 是           | 生成签名的时间戳                                             |
| 11       | signature   | 是           | 请使用接入方私钥签名，输出格式为base64.                      |

 

响应报文参数说明

| **序号** | **参数名** | **是否必填** | **备注**                             |
| -------- | ---------- | ------------ | ------------------------------------ |
| 1        | resp       |              | 成功通知以后发送 {‘resp’: ‘00’} 返回 |

#### 5.8.4.4 报文示例

```java
请求报文：
{
        "appId": "99eb12************************a01ba",
        "qualNum": "681ee844-40de-0376-1257-3ace7bc3a7c2",
        "qualType": "mobile",
        "qualValue": "tPAcG+FFZu4=",
        "count": "tPAcG+FFZu4=",
        "transNumber": "12321334453254334",
        "startDate": "20191029",
        "endDate": "20191031",
        "nonceStr": "C66S9qvz21V84cOT",
        "timestamp": "1572337564",
        "signature": "IXd9vYPMEAa0ZJ4/3A5aOmvyb0aOaGe+6PqH7wU7VZkTiKoI0vTTwSid5BhEXWv7foKpuGvL0vMxUovfbVPV+KrJsD6rRYKRxJ0gKswj72sZWhuU0nvDCclAlwB+e4Y7qTE7N9ufjJ1EjgFTQdDgAiyhIKvMgXEL7rh8I+oA3fM="}
响应报文：
{
	"msg": "",
        "params": {
            "resp": ""
        },
        "resp": "00"
}

```

 

### 5.8.5 抽奖（红包/票券）<qual.reduce>

#### 5.8.5.1 功能描述

接入方通过云闪付抽奖（红包/票券）接口，给用户赠送（非定额/定额）红包或者商城券。

#### 5.8.5.2 请求地址

https://open.95516.com/open/access/1.0/qual.reduce

#### 5.8.5.3 报文结构

请求报文参数说明

| **序号** | **参数名**     | **是否必填** | **备注**                                                  |
| -------- | -------------- | ------------ | --------------------------------------------------------- |
| 1        | appId          | 是           | 接入方的唯一标识                                          |
| 2        | activityNumber | 是           | 活动 编号                                                 |
| 3        | orderAmount    | 否           | 订单金额                                                  |
| 4        | qualNum        | 是           | 资格池编号                                                |
| 5        | qualType       | 是           | 资格类型  固定值“open_id”、“mobile”、“card_no”            |
| 6        | qualValue      | 是           | 资格值  （使用symmetricKey 对称加密，内容为base64格式）   |
| 7        | certId         | 否           | 身份证号  （使用symmetricKey 对称加密，内容为base64格式） |
| 8        | icTerminal     | 否           | 设备终端                                                  |
| 9        | qrCode         | 否           | 红包码                                                    |
| 10       | transNumber    | 是           | 流水号（确保唯一）                                        |
| 11       | nonceStr       | 是           | 生成签名的随机串                                          |
| 12       | timestamp      | 是           | 生成签名的时间戳                                          |
| 13       | signature      | 是           | 请使用接入方私钥签名，输出格式为base64.                   |

 

响应报文参数说明

| **序号** | **参数名**  | **是否必填** | **备注**                             |
| -------- | ----------- | ------------ | ------------------------------------ |
| 1        | resp        |              | 成功通知以后发送 {‘resp’: ‘00’} 返回 |
| 2        | respTime    |              | 结果时间 yyyyMMddHHmmss              |
| 3        | awardInfo   |              | 奖项信息   内容字段如下6.6.5.4表格   |
| 4        | transNumber |              | 订单流水号                           |

 awardInfo信息

| **数据名称**    | **中文名称**             | **选填/****必填** | **说明**                              |
| --------------- | ------------------------ | ----------------- | ------------------------------------- |
| activityNumber  | 活动编号                 | 选填              |                                       |
| activityName    | 活动名称                 | 选填              |                                       |
| beginTime       | 活动开始时间             | 选填              |                                       |
| endTime         | 活动结束时间             | 选填              |                                       |
| awardId         | 奖项Id                   | 选填              |                                       |
| awardType       | 奖项类型                 | 选填              | 06：红包  07：商城券                  |
| awardName       | 奖项名称                 | 选填              |                                       |
| awardValue      | 奖项值                   | 选填              | 红包为红包金额，单位:分  商城券为张数 |
| extAcctId       | 外部账户id               | 选填              | 商城券为券ID                          |
| extAcctName     | 外部账户名称             | 选填              | 商城券为券名称                        |
| drawDesc        | 中奖说明                 | 选填              |                                       |
| couponStartDate | 商城券生效开始时间       | 选填              |                                       |
| couponEndDate   | 商城券生效结束时间       | 选填              |                                       |
| couponGoodsUrl  | 商城券对应商品的链接地址 | 选填              |                                       |

说明：                                        

除了signature之外，请求参数中所有字段参与验签。所有待签名参数（字符编码为 UTF-8）按照字段名的ASCII 码从小到大排序（字典序）后，使用键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。对拼接字符串string1进行RSA签名后，按base64格式出现在报文中。

 

#### 5.8.5.4 报文示例

```java
请求报文：
{
	"appId": "99eb12************************a01ba",
        "activityNumber": "1320190910000374",
        "orderAmount": "1",
        "qualNum": "29fd5a16-77ae-0842-750d-aed31e1830bd",
        "qualType": "mobile",
        "qualValue": "YRpIAfdvpf17mlI5jdxr7A==",
        "certId": "FzBGmOPyOwD4GyztXlqx74J5DKFX8NRB",
        "icTerminal": "leirwei",
        "qrCode": "33333",
        "transNumber": "9865785412639875641235874156728",
        "nonceStr": "JuVE7wEdzY7sBym3",
        "timestamp": "1572337528",
        "signature": "DviHal9IayNLGISvTPLq1L4SwXPjxPgMGV6QFso4UTEXHhTHrQCNI6tCpySGIZ7JTQYEn7Vzr7tDHpeNeYhps9NSyGoxAu+s+V4erLP/sxtZMSTxu2Vd8HURzkHYP+MbutTfk18iYF5D9HMI3sxB6/B6mzwsdwSs1i4NkCPJAGk="}
响应报文：
{
 "msg": "",
        "params": {
            "resp": "",
            "respTime": "",
            "awardInfo": {
                "activityNumber": "",
                "activityName": "",
                "beginTime": "",
                "endTime": "",
                "awardId": "",
                "awardType": "",
                "awardName": "",
                "awardValue": "",
                "extAcctId": "",
                "extAcctName": "",
                "drawDesc": "",
                "couponStartDate": "",
                "couponEndDate": "",
                "couponGoodsUrl": ""
            },
            "transNumber": ""
        },
        "resp": "00"
}

```

 

# 6 前台插件UPSDK

## 6.1 接口清单

| **序号** | **类型** | **接口代码**                         | **接口名称**                                                 |
| -------- | -------- | ------------------------------------ | ------------------------------------------------------------ |
| 1.       | SDK      | pay                                  | 支付，银联在线支付                                           |
| 2.       | SDK      | addBankCard                          | 绑定银行卡                                                   |
| 3.       | SDK      | setNavigationBarTitle                | 设置标题栏标题                                               |
| 4.       | SDK      | setNavigationBarRightButton          | 设置标题栏右边按钮                                           |
| 5.       | SDK      | closeWebApp                          | 关闭当前WEB窗口                                              |
| 6.       | SDK      | showFlashInfo                        | 显示toast文言                                                |
| 7.       | SDK      | scanQRCode                           | 启动云闪付扫码                                               |
| 8.       | SDK      | chooseImage                          | 拍照或从手机相册中选图                                       |
| 9.       | SDK      | getLocationCity                      | 返回用户在首页选取的城市                                     |
| 10.      | SDK      | verifyPayPwd                         | 进入支付密码输入页面，验证支付密码输入正确与否               |
| 11.      | SDK      | getLocationGps                       | 获取用户当前GPS点，基于高德火星坐标系                        |
| 12.      | SDK      | showSharePopup                       | 显示分享弹框                                                 |
| 13.      | SDK      | chooseFileFromAlbum                  | 选择本地音视频（或图片），获取到文件基本信息                 |
| 14.      | SDK      | readAlbumData                        | 将选取的文件分块获取文件流的base64                           |
| 15.      | SDK      | startAudioRecording                  | 开始录音                                                     |
| 16.      | SDK      | stopAudioRecording                   | 停止录音                                                     |
| 17.      | SDK      | readAudioRecordingData               | 将录好的本地音频文件分块获取文件流的base64                   |
| 18.      | SDK      | startVideoRecording                  | 将录好的本地视频文件分块获取文件流的base64                   |
| 19.      | SDK      | readFaceVideoData                    | 分块获取活体检测后保存在本地的视频文件流的base64             |
| 20.      | SDK      | readFaceImageData                    | 分块获取活体检测后保存在本地的图片文件流的base64             |
| 21.      | SDK      | getScreenBrightness                  | 获取当前屏幕亮度                                             |
| 22.      | SDK      | setScreenBrightness                  | 设置屏幕亮度                                                 |
| 23.      | SDK      | monitorScreenShot                    | 监听屏幕截屏（仅限ios使用）                                  |
| 24.      | SDK      | navi                                 | 地图导航插件                                                 |
| 25.      | SDK      | openBluetoothAdapter                 | 初始化蓝牙                                                   |
| 26.      | SDK      | closeBluetoothAdapter                | 关闭蓝牙                                                     |
| 27.      | SDK      | getBluetoothAdapterState             | 获取蓝牙状态                                                 |
| 28.      | SDK      | startBluetoothDevicesDiscovery       | 开始扫描蓝牙状态                                             |
| 29.      | SDK      | stopBluetoothDevicesDiscovery        | 停止扫描蓝牙设备                                             |
| 30.      | SDK      | getBluetoothDevices                  | 获取已扫描到的设备列表                                       |
| 31.      | SDK      | connectBLEDevice                     | 连接低功耗蓝牙设备                                           |
| 32.      | SDK      | disconnectBLEDevice                  | 断开低功耗设备的蓝牙连接                                     |
| 33.      | SDK      | getConnectedBluetoothDevices         | 获取已连接的设备列表                                         |
| 34.      | SDK      | getBLEDeviceServices                 | 获取设备的服务列表                                           |
| 35.      | SDK      | getBLEDeviceCharacteristics          | 获取蓝牙设备的所有characteristic                             |
| 36.      | SDK      | writeBLECharacteristicValue          | 向蓝牙设备写数据                                             |
| 37.      | SDK      | readBLECharacteristicValue           | 向蓝牙设备读数据                                             |
| 38.      | SDK      | notifyBLECharacteristicValueChange   | 开启蓝牙设备notify提醒功能                                   |
| 39.      | SDK      | registerBluetoothDeviceFound         | 注册发现新设备监听                                           |
| 40.      | SDK      | cancelBluetoothDeviceFound           | 取消发送新设备监听                                           |
| 41.      | SDK      | registerBLEConnectionStateChange     | 注册蓝牙连接状态监听                                         |
| 42.      | SDK      | cancelBLEConnectionStateChange       | 取消蓝牙连接状态监听                                         |
| 43.      | SDK      | registerBLECharacteristicValueChange | 注册特征值变化监听                                           |
| 44.      | SDK      | cancelBLECharacteristicValueChange   | 取消特征值变化监听                                           |
| 45.      | SDK      | registerBluetoothAdapterStateChange  | 注册蓝牙状态变化监听                                         |
| 46.      | SDK      | cancelBluetoothAdapterStateChange    | 取消蓝牙状态变化监听                                         |
| 47.      | SDK      | openBluetoothSetting                 | 跳转到手机系统的蓝牙设置                                     |
| 48.      | SDK      | getHCEState                          | 判断当前设备是否支持HCE能力                                  |
| 49.      | SDK      | startHCE                             | 调用startHCE初始化本机NFC模块                                |
| 50.      | SDK      | stopHCE                              | 调用stopHCE停止手机的NFC模块                                 |
| 51.      | SDK      | onHCEMessage                         | 调用onHCEMessage监听芯片响应的消息事件中返回。               |
| 52.      | SDK      | sendHCEMessage                       | 调用 sendHCEMessage发送写卡数据并在成功回调中接收芯片响应的消息 |
| 53.      | SDK      | openNFCSetting                       | 打开NFC设置页面                                              |

 

## 6.2 UPSDK使用步骤

步骤一：引入JS文件

在需要调用JS接口的页面引入如下JS文件（仅支持https）： 

https://open.95516.com/s/open/js/upsdk.js

 

注意：

Ø upsdk.js文件依赖Zepto或Jquery。

Ø Zepto的版本要求1.0及以上，Jquery的版本要求1.4及以上（建议用最新的Zepto及Jquery）。

 

步骤二：通过config接口注入权限验证配置

所有需要使用UPSDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，如果跳转的页面无需使用插件，则无需config，否则需要重新执行config）。

```javascript
//初始化示例代码
upsdk.config({
	appId:' ',   //必填，接入方的唯一标识，由云闪付分配
	timestamp:' ' ,   //必填，生成签名的时间戳，从1970年1月1日00:00:00至今的秒数，取东八区的北京时间，例1556096162
	nonceStr: ' ',    //必填，生成签名的随机串，参见附录三生成签名随机字符串
	signature: ' ',   //必填，生成签名的摘要，参见附录一、附录二
	debug: true   //开发阶段可打开此标记，云闪付APP会将调试信息toast出来
});

```

建议接入方在开发联调时，打开debug: true 开关，UPSDK只有在开关打开时才会输出状态信息，帮助开发者定位错误。 请务必在最终生产版本关闭此开关。

 

**步骤三：配置信息验证**

通过ready接口处理成功验证

```javascript
upsdk.ready(function(){
//config信息验证后会执行ready方法         
});
```

通过error接口处理失败验证

```javascript
upsdk.error(function(err){
//config信息验证失败会执行error方法  
});

```

 

## 6.3 支付接口<pay>

```javascript
upsdk.pay({
    tn: '支付流水号Ticket Number.',
    success: function(){
        // 支付成功，开发者执行后续操作
    },
    fail: function(err){
        // 支付失败，err.msg 是失败原因描述，比如TN号不合法，或者用户取消了交易等等
    }
});

```

前台支付对接，交易TN号获取方式参见如下URL：

https://open.unionpay.com/tjweb/acproduct/APIList?acpAPIId=393&&apiservId=450

整个支付流程详解请参见附录五-支付流程详解。

## 6.4 拍照或从手机相册中选图接口<chooseImage>

```javascript
upsdk.chooseImage({
    maxWidth: '目标图片宽度，默认500',      //可选。
    maxHeight: '目标图片高度，默认1000',    //可选。
    sourceType: '1|2|3，仅允许拍照|仅允许从手机相册中选图|拍照或从手机相册中选图都支持， 默认为3',   //可选
    success: function (data) {
        if (data.base64) {
            // 目标图片采用base64编码
        }
    }
});

```

此接口还会返回选择的图片类型，如jpg|png|gif

## 6.5 设置页面标题接口<setNavigationBarTitle>

```javascript
upsdk.setNavigationBarTitle({
    title: '设置云闪付标题'
});
```

## 6.6 设置标题栏右边按钮接口<setNavigationBarRightButton>

```javascript
upsdk.setNavigationBarRightButton({
   // title和image可任选其一，同时有的话，title优先
   title: '标题栏文字',
   image: ' 按钮图片绝对路径',­­­­­­
   handler: function(){
     // 用户点击标题按钮以后回调函数
   }
 });
```

支持文字、图片

## 6.7 调用云闪付的扫码功能接口<scanQRCode>

```javascript
upsdk.scanQRCode({
   scanType: ["qrCode","barCode"],
   success: function(result){
     alert('Scan result = ' + result);
   }
 });
```

## 6.8 绑定银行卡接口<addBankCard>

```javascript
upsdk.addBankCard({
   success: function(){
     // 绑卡成功
   },
   fail: function(){
     // 绑卡失败或者用户取消
   }
   sence: '场景号' //可选参数，用来控制添加特定类型的银行卡，例如只添加信用卡
 });
```



## 6.9 Toast文言显示接口<showFlashInfo>

```javascript
upsdk.showFlashInfo({
   msg: '绑定银行卡成功'
 });
```

upsdk.showFlashInfo({
   msg: '绑定银行卡成功'
 });

## 6.10 关闭当前WEB窗口接口<closeWebApp>

```javascript
upsdk.closeWebApp();
```

## 6.11 返回用户在首页选取的城市<getLocationCity>

```javascript
upsdk.getLocationCity({
   success: function(cityCd){

	//插件返回的城市代码cityCd 符合全国行政区划代码表标准，比如广州市为440100  

});
```

## 6.12 用户通过输入支付密码授权<verifyPayPwd>

```javascript
upsdk.verifyPayPwd({
   transNo: '后台获取的支付密码验证流水号', 

bizType: '业务类型编号',
   success: function() {
     // 支付密码验证通过
   }
 });
```

transNo：通过后台接口“获取支付密码验证流水号<verifyPwdSeqId>”获取

bizType：业务类型，请与云闪付业务方确定

## 6.13 获取用户当前位置经纬度<getLocationGps>

```javascript
upsdk.getLocationGps({
   success: success,
   fail: fail
 });
```

成功返回对象，里面含有属性latitude 和longitude，基于高德地图坐标系

## 6.14 云闪付分享插件<showSharePopup>

```javascript
upsdk.showSharePopup({
   title: '银联云闪付随机立减大优惠～！',
   desc: '我刚刚使用银联云闪付，省了30元，大家快来使用吧。 ',
   shareUrl: 'https://youhui.95516.com',
   picUrl: 'https://youhui.95516.com/web/image/mchnt/coupon/Z00000000138804_logo_list_web.jpg'
 });
```

picUrl 选填，默认显示银联云闪付图标

## 6.15 获取本地音视频或图片<chooseFileFromAlbum & readAlbumData>

**步骤一：选择本地音视频或图片，获取文件基本信息**

```javascript
upsdk.chooseFileFromAlbum({
    maxSize:'1024',    //最大上传的文件大小（单位byte），若选择的文件大于该值进入失败回调。建议值不要超过100m（即100*1024*1024）
    sourceType:'00',   //00:仅支持视频文件，例如mp4、mov等。01：仅支持图片文件，例如png、jpg等。02：支持视频+图片文件。
    success:function(data){  
		// 成功返回{url:'视频地址',size:'视频大小',filename:'视频文件名，例如123.mp4'}
	}, 
    fail: function(err){
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
	}
});

```

 

**步骤二：将选取的文件分块，通过以下插件分块获取文件流的****base64**

```javascript
upsdk.readAlbumData({
    url: 'url',   //即步骤一成功回调获取视频地址url
    bufferSize: '1024',   //分块大小，单位byte，最大不可以超过512kb（即512*1024）
    fromOffset: '0',     //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推
success:function(data){  
		// 成功返回{data:'文件块的base64',isFinished:'1'}
		// isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕
}, 
fail: function(err){  
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
    }
});

```



## 6.16 录音并获取录音文件< startAudioRecording & stopAudioRecording & readAudioRecordingData>

**步骤一：开始录音，获得成功回调后，可以选择主动停止录音，或者等待最大录音时间后自动停止。**

```javascript
upsdk.startAudioRecording({
    maxTime:'10',   //单位为秒，必选参数，允许用户录制的最大时间
    success:function(){  
		//成功回调，此处可选择主动停止录音，或者等待maxTime
}, 
    fail: function(err){
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
	}
});
upsdk.stopAudioRecording({
    success:function(){  
		// 成功回调，此处可获取已录好的本地音频文件
}, 
    fail: function(err){
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
	}
});

```

 

**步骤二：将录好的本地音频文件分块，通过以下插件分块获取文件流的****base64****。【注意，该插件应该在停止录音插件成功回调中使用，或者等待最大录音时间****maxTime****后才能使用】**

```javascript
upsdk.readAudioRecordingData({
    bufferSize: '1024',  //分块大小，单位byte，最大不可以超过512kb（即512*1024）
    fromOffset: '0',    //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推
	success:function(data){  
		// 成功返回{data:'文件块的base64',isFinished:'1'}
		// isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕
	}, 
    fail: function(err){  
   		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
    }
});

```

备注：始终保存最后一次录音的文件，仅在卸载APP后才会清除。

## 6.17 录视频并获取视频文件<startVideoRecording & readVideoRecordingData>

步骤一：开始录视频，到达最大录制时间，视频自动录制结束。用户录制完视频后调用成功回调。

```javascript
upsdk.startVideoRecording({
    maxTime: '10',   //单位为秒，必选参数，允许用户录制的最大时间
    success:function(){  
		// 成功回调，此时用户已完成录制视频，可获取录制的视频文件
	}, 
    fail: function(err){
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
	}
});

```

 步骤二：将录好的本地视频文件分块，通过以下插件分块获取文件流的base64。【注意，该插件应该在开始录视频插件成功回调中使用。】

```javascript
upsdk.readVideoRecordingData({
    bufferSize: '1024',   //分块大小，单位byte，最大不可以超过512kb（即512*1024）
    fromOffset: '0',     //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推
	success:function(data){  
		// 成功返回{data:'文件块的base64',isFinished:'1'}
		// isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕
	}, 
    fail: function(err){  
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'02',msg:'权限失败'}
    		// {code:'03',msg:'用户选择的文件超过最大值'}
    		// {code:'04',msg:'其他错误'}
    }
});

```

备注：始终保存最后一次录音的文件，仅在卸载APP后才会清除。

## 6.18 获取活体检测后保存在本地的视频和图片<readFaceVideoData& readFaceImageData>

获取活体检测的视频（分块获取，视频格式为mp4）

```javascript
upsdk.readFaceVideoData({
success:function(data){  
		// 成功返回{data:'文件块的base64',isFinished:'1'}
		// isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕
	}, 
    fail: function(err){  
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'07', msg:'文件不存在'}
    }
});

```

 

获取活体检测的图片（分块获取，图片格式为jpg）

```javascript
upsdk.readFaceImageData({
	success:function(data){  
		// 成功返回{data:'文件块的base64',isFinished:'1'}
		// isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕
	}, 
    fail: function(err){  
    		// 失败回调{code:'失败码',msg:'失败原因描述'}
		// {code:'00',msg:'参数错误'}
    		// {code:'01',msg:'内部错误'}
    		// {code:'07', msg:'文件不存在'}
    }
});

```



## 6.19 获取当前屏幕亮度<getScreenBrightness>

```javascript
upsdk.getScreenBrightness({
    success:function(data){  
    // 成功回调 {data.brightness} 
    }, 
    fail: function(msg){  
    // 失败回调
    }
});

```



## 6.20 设置屏幕亮度<setScreenBrightness>

```javascript
upsdk.setScreenBrightness({
    brightness: '0.2',     // 屏幕亮度值，范围取值0-1。精确到小数点后一位
    success:function(data){  
    // 成功回调 {data.brightness} 
    }, 
        fail: function(msg){  
    // 失败回调
    }
});

```



## 6.21 监听屏幕截屏（仅限ios使用）<monitorScreenShot>

```javascript
upsdk.monitorScreenShot({
    success:function(){  
    // 成功回调，表明用户已经进行了截屏
    }
});

```

备注：通常只在需要监听的页面调用该插件打开监听。

## 6.22 地图导航插件<navi>

```javascript
upsdk.navi({
    sLat:'31.23958',        // 起点纬度
	sLon:'121.499763',     // 起点经度
    sName:'上海东方明珠',  // 起点名称
    dLat:'39.917854',       // 终点维度
    dLon:'116.397006',     // 终点经度
    dName:'北京故宫',     // 终点名称
    success:function(){  
    // 成功回调 
    }, 
        fail: function(msg){  
    // 失败回调
    }
});

```

 

## 6.23  蓝牙插件

### 6.23.1初始化蓝牙<openBluetoothAdapter>

```javascript
 upsdk.openBluetoothAdapter({
    success:function(data){  
    // 成功回调 {"isSupportBLE": "yes"} 支持BLE，不区分大小写
    // 成功回调 {"isSupportBLE": "no"} 不支持BLE，不区分大小写
    }, 
        fail:function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

注意：只有初始蓝牙成功后才能正常执行后续操作蓝牙功能

### 6.23.2关闭蓝牙<closeBluetoothAdapter>

```javascript
 upsdk.closeBluetoothAdapter({
    success:function(){  
    // 成功回调 
    }, 
        fail: function(){  
       // 失败回调
    }
});

```

 

### 6.23.3获取蓝牙状态<getBluetoothAdapterState>

```javascript
 upsdk.getBluetoothAdapterState({
    success:function(data){  
    // 成功回调 {"available":"yes","discovering":"no"} 
    // available的值为yes 或者 no，不区分大小，表明当前蓝牙是否可用
    // discovering 的值为 yes 或者 no，不区分大小，表明是否在扫描蓝牙中
    }, 
        fail: function(){  
       // 失败回调
    }
});j

```



### 6.23.4开始扫描蓝牙设备<startBluetoothDevicesDiscovery>

```javascript
upsdk.startBluetoothDevicesDiscovery({
    services: [‘具体的uuid’]，   // 可选，蓝牙设备主 service 的 uuid 列表
    allowDuplicatesKey: ‘yes’,   // 可选，值为yes或no，是否允许重复上报同一设备
    success:function(){  
    // 成功回调 
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

注意：可选字段不赋予值的时候不要上送

 

### 6.23.5停止扫描蓝牙设备<stopBluetoothDevicesDiscovery>

 

```javascript
upsdk.stopBluetoothDevicesDiscovery({
    services: [‘具体的uuid’]，   // 可选，蓝牙设备主 service 的 uuid 列表
    allowDuplicatesKey: ‘yes’,   // 可选，值为yes或no，是否允许重复上报同一设备
    success:function(){  
    // 成功回调 
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});
```

注意：可选字段不赋予值的时候不要上送

 

### 6.23.6获取已扫描到的设备列表<getBluetoothDevices>

```javascript
upsdk.getBluetoothDevices({
    success:function(data){  
    // 成功回调 {devices: [{device}]}，device参数如下表所示
    }, 
        fail: function(){  
       // 失败回调 
    }
});

```

![1615446530883](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446530883.png)

 

### 6.23.7连接低功耗蓝牙设备<connectBLEDevice>

```javascript
 upsdk.connectBLEDevice({
    deviceId:’’,  // 必填，设备id，可从获取已扫描到的设备列表6）中获取该值，并进行连接
    success:function(){  
    // 成功回调 
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```



### 6.23.8断开低功耗蓝牙设备<disconnectBLEDevice>

```javascript
upsdk.disconnectBLEDevice({
    deviceId:’’,  // 必填，设备id，可从获取已扫描到的设备列表中获取该值，并进行连接
    success:function(){  
    // 成功回调 
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.9获取已连接的设备列表<getConnectedBluetoothDevices>

注意:­­­­只有当监听蓝牙连接状态<registerBLEConnectionStateChange>执行了回调，且连接状态connected为yes时，才可获取到设备列表

可选字段不赋予值的时候不要上送

```javascript
upsdk.getConnectedBluetoothDevices({
    services: [‘具体的uuid’]，   // 可选，蓝牙设备主 service 的 uuid 列表
    success:function(data){  
    // 成功回调 {devices: [{device}]}，device参数同 6) 
    }, 
        fail: function(){  
       // 失败回调 
    }
});

```



### 6.23.10获取设备的服务列表<getBLEDeviceServices>

```javascript
upsdk.getBLEDeviceServices ({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    success:function(data){  
    // 成功回调 {"services": [{service}]}，service参数如下表所示
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

![1615446547131](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446547131.png)

### 6.23.11获取蓝牙设备的所有Cahracteristic<getBLEDeviceCharacteristics>

```javascript
 upsdk.getBLEDeviceCharacteristics({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    serviceId:’’,  // 必填，服务id，需从获取设备的服务列表 10）中获取该值
    success:function(data){  
    // 成功回调 {"characteristics": [{characteristic}]}，characteristic参数如下表所示
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

![1615446560514](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446560514.png)

注意：确保该接口能正常获取返回的码表信息，如不能将影响后续系列操作；

 

### 6.23.12向蓝牙设备写数据<writeBLECharcateristicValue>

注意：­­­­调用该插件时，需characteristicId对应的属性有Write权限 

```javascript
upsdk.writeBLECharacteristicValue ({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    serviceId:’’,  // 必填，服务id，需从获取设备的服务列表 10）中获取该值
    characteristicId:’’,  // 必填，特征id，需从获取设备的特征列表 11）中获取该值
    value:’’,  // 必填，写的数据 
    success:function(){  
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```



### 6.23.13向蓝牙设备读数据<readBLECharacteristicValue>

注意：­­­­调用该插件时，需characteristicId对应的属性有Read权限 

```javascript
upsdk.readBLECharacteristicValue({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    serviceId:’’,  // 必填，服务id，需从获取设备的服务列表 10）中获取该值
    characteristicId:’’,  // 必填，特征id，需从获取设备的特征列表 11）中获取该值
    success:function(data){  
       // 成功回调{characteristic: {characteristicId:’’, serviceId:’’, value:’’}}
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```



### 6.23.14开启蓝牙设备notify提醒功能<notifyBLECharacteristicValueChange>

注意：­­­­调用该插件时，需characteristicId对应的属性有Notify或Indicate权限 

```javascript
upsdk.notifyBLECharacteristicValueChange({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    serviceId:’’,  // 必填，服务id，需从获取设备的服务列表 10）中获取该值
    characteristicId:’’,  // 必填，特征id，需从获取设备的特征列表 11）中获取该值
    state:’’,  //选填，预留参数，暂无作用；只能开启特征值支持的模式；
                 //如果特征值支持notify和indicate两种模式，则使用notify模式
    success:function(){  
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.15注册发现新设备监听<registerBluetoothDeviceFound>

```javascript
upsdk.notifyBLECharacteristicValueChange({
    deviceId:’’,  // 必填，设备id，需从获取已连接的设备列表 9）中获取该值
    serviceId:’’,  // 必填，服务id，需从获取设备的服务列表 10）中获取该值
    characteristicId:’’,  // 必填，特征id，需从获取设备的特征列表 11）中获取该值
    state:’’,  //选填，预留参数，暂无作用；只能开启特征值支持的模式；
                 //如果特征值支持notify和indicate两种模式，则使用notify模式
    success:function(){  
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.16取消发现新设备监听<cancelBluetoothDeviceFound>

```javascript
upsdk.cancelBluetoothDeviceFound({
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.17注册蓝牙连接状态监听<registerBLEConnectionStateChange>

```javascript
upsdk.registerBLEConnectionStateChange ({
    callback:function(state){  
       // 异步，设备连接状态发生改变时会执行该回调,
       // 并返回参数state = {
       //                        deviceId:’’,  // 设备id
       //                        connected:’’  //值为yes表示已连接，值为no表示未连接
       //                     }
    }, 
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.18取消蓝牙连接状态监听<cancelBLEConnectionStateChange>

```javascript
upsdk.cancelBLEConnectionStateChange({
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

### 6.23.19注册特征值变化监听<registerBLECharacteristicValueChange>

```javascript
upsdk.registerBLECharacteristicValueChange({
    callback:function(characteristic){  
       // 异步，特征值发生变化时会执行该回调,
       // 并返回参数characteristic = {
       //         deviceId:’’,
       //         serviceId:’’,
       //         characteristicId:’’,
       //         value:’’
       //    }
    }, 
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
	}
});

```

 

### 6.23.20取消特征值变化监听<cancelBLECharacteristicValueChange>

```javascript
upsdk.cancelBLECharacteristicValueChange({
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```



### 6.23.21注册蓝牙状态变化监听<registerBluetoothAdapterStateChange>

```javascript
upsdk.registerBluetoothAdapterStateChange({
    callback:function(data){  
       // 异步，设备从未初始化->初始化 或 从初始化->未初始化 等状态发生变化时，执行回调
       // data = {state:’’}   state取值说明见下表
    }, 
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

![1615446579486](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446579486.png)

 

### 6.23.22取消取消蓝牙变化监听<cancelBluetoothAdapterStateChange>

```javascript
upsdk.cancelBluetoothAdapterStateChange({
    success:function(){  // 插件执行成功
    }, 
        fail: function(data){  
       // 失败回调 {code:’’,msg:’’}
    }
});

```

 

### 6.23.23跳转到手机系统的蓝牙设置<openBluetoothSetting>

```javascript
upsdk.openBluetoothSetting({
    success:function(){  // 插件执行成功
    }, 
        fail: function(){  
    }
});
```

备注：错误码说明

![1615446593554](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446593554.png)

 

## 6.24  NFC插件（HCE模式）

### 6.24.1判断当前设备是否支持HCE能力<getHCEState>

```javascript
upsdk.getHCEState ({
    success:function(data){  // 插件执行成功
       //
    }, 
        fail: function(){  
    }
});

```

成功回调的响应：

| code | msg                                  |
| ---- | ------------------------------------ |
| 0000 | ok                                   |
| 1001 | 当前设备不支持NFC                    |
| 1002 | 当前设备支持NFC，但系统NFC开关未开启 |
| 1003 | 当前设备支持NFC，但不支持HCE         |
| 1004 | 未设置云闪付为默认NFC应用            |

 

 

### 6.24.2调用startHCE初始化本机NFC模块<startHCE>

```javascript
upsdk.startHCE ({

    aidList:"",// 必填，需要注册到系统的 AID 列表
    success:function(data){  // 插件执行成功
       //
    }, 
        fail: function(){  
    }
});

```



```
注释：aidList 参数类型为string 多个AID 请用“,”隔开,AID长度限制是10-32位由0-9和A-F的字符组成实例 "A000000010,A123456789"
```

 

| code | msg                 |
| ---- | ------------------- |
| 0000 | ok                  |
| 1101 | 注册AID失败         |
| 1102 | AID列表参数格式错误 |
| 1103 | 输入参数错误        |

 

### 6.24.3调用stoptHCE停止手机NFC模块<stoptHCE>

```javascript
upsdk.stopHCE({
    success:function(data){  // 插件执行成功
       //
    }, 
        fail: function(){  
    }
});

```



| code | msg          |
| ---- | ------------ |
| 0000 | ok           |
| 1401 | 输入参数错误 |
| 1402 | 注销AID失败  |

 

 

### 6.24.4调用onHCEMessage监听芯片响应的消息事件中返回<onHCEMessage>

```javascript
upsdk.onHCEMessage({
    success:function(data){  // 插件执行成功
       //
    }, 
        fail: function(){  
    }
});

```

注释：该方法用于监听NFC设备发来的指令，调用该方法需要调用startHCE成功后才可以调用

| code | msg          |
| ---- | ------------ |
| 0000 | ok           |
| 1301 | 发送指令失败 |

 

### 6.24.5调用sendHCEMessage发送写卡数据并在成功回调中接口芯片响应的消息<sendHCEMessage>

```javascript
upsdk.sendHCEMessage({
    data:"",// 必填，命令数据，类型是string，内容是01组成的2进制数据
    success:function(data){  // 插件执行成功
       //
    }, 
        fail: function(){  
    }
});
```

### 6.24.6打开NFC设置页面<openNFCSetting>

```javascript
upsdk.openNFCSetting({
 
success:function(data){  // 插件执行成功
   //
}, 
    fail: function(){  
}
});
 
```



# 7 附录

## 7.1 附录一：UPSDK初始化签名因子



Ø appId=a5949221470c4059b9b0b45a90c81527

Ø nonceStr=Wm3WZYTPz0wzccnW

Ø timestamp=1414587457

Ø url=http://mobile.xxx.com?params=value

Ø frontToken=U72eJp21SkuzKRdUK+jyFw== //通过后台接口”获取frontToken<frontToken>”获取

 

注意:



## 7.2 附录二：SHA256签名

​    步骤一：拼接待签名字符串，得到string1

对所有待签名参数按照字段名的ASCII码进行从小到大排序（字典序），然后使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1，如：

appId=a5949221470c4059b9b0b45a90c81527&frontToken=U72eJp21SkuzKRdUK+jyFw==&nonceStr=Wm3WZYTPz0wzccnW&timestamp=1414587457&url=http://mobile.xxx.com?params=value

​    

​    步骤二：对待签名字符串进行SHA256签名，得到signature

​    将步骤一得到的待签名字符串string1转换成byte数组，传入方法sha256(byte[] data)中，执行后将返回签名结果signature。

如：a4bb34a2b60aa34ec4f03754547ca3e39a80e628b9760323d10561997935bb42。

 

JAVA实现代码：

```java
public static String sha256(byte[] data) { 
    try { 
        MessageDigest md = MessageDigest.getInstance("SHA-256"); 
        return bytesToHex(md.digest(data)); 
        } catch (Exception ex) { 
        logger.info("Never happen.", ex); 
        return null; 
    } 
}

public static String bytesToHex(byte[] bytes) { 
    String hexArray = "0123456789abcdef"; 
    StringBuilder sb = new StringBuilder(bytes.length * 2); 
    for (byte b : bytes) { 
        int bi = b & 0xff; 
        sb.append(hexArray.charAt(bi >> 4)); 
        sb.append(hexArray.charAt(bi & 0xf)); 
    } 
    return sb.toString(); 
}
	

```

​    

​    PHP实现代码：

```php
public function sha256($data, $rawOutput = false){
    if (!is_scalar($data)) {
        return false;
    }
    $data = (string)$data;
    $rawOutput = !!$rawOutput;
    return hash('sha256', $data, $rawOutput);
}
```

注意：

Ø 签名用的nonceStr和timestamp必须与请求参数中的nonceStr和timestamp相同。

## 7.3 附录三：生成签名随机字符串nonceStr

 JAVA实现代码

```java
public static String createNonceStr(){
	String sl = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    StringBuilder sb = new StringBuilder();
    for(int i = 0 ; i < 16 ; i ++){
        sb.append(sl.charAt(new Random().nextInt(sl.length())));
    }
    return sb.toString();
}

```

PHP实现代码

```php
public function createNonceStr($length = 16) {
    $str = null;
    $strPol = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz";
    $max = strlen($strPol)-1;
    for($i=0;$i<$length;$i++){
        $str.=$strPol[rand(0,$max)]; //rand($min,$max)生成介于min和max两个数之间的一个随机整数
    }
    return $str;
}

```

 

## 7.4 附录四：Base64 编码转换代码

JAVA实现代码：

```java
public static byte[] fromBase64(String p_Str) throws IOException{
    byte[] byteBuffer = new BASE64Decoder().decodeBuffer(p_Str);
    return byteBuffer;
}

```

​    PHP实现代码

```php
base64_encode($str);
```

## 7.5 附录五：银联公钥upPublicKey验证签名

```java
boolean result = verify(params, upPublicKey, signature);
	public static boolean verify(Map<String, String> param, String verifyKey, String sign) throws Exception {
		String value = sortMap(param);
		byte[] keyBytes = Base64.decodeBase64(verifyKey);
		X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyf = KeyFactory.getInstance("RSA");
        PublicKey pubkey = keyf.generatePublic(keySpec);
        Signature signature = Signature.getInstance("SHA256WithRSA");
		signature.initVerify(pubkey);
		signature.update(value.getBytes());
		boolean result = signature.verify(Base64.decodeBase64(sign.getBytes()));
		return result;
	}
	
	public static String sortMap(Map<String, String> param){
		StringBuilder result = new StringBuilder();
		Collection<String> keySet = param.keySet();
		List<String> list = new ArrayList<String>(keySet);
		Collections.sort(list);
		for (int i = 0; i < list.size(); ++i) {
			String key = list.get(i);
			if("symmetricKey".equals(key)){
				continue;
			}
			if(param.get(key) == null || "".equals(param.get(key).trim())){
				continue;
			}
			result.append(key).append("=").append(param.get(key)).append("&");
		}
		return result.substring(0, result.length() - 1);
	}

```

​    注意

Ø Base64使用org.apache.commons.codec.binary.Base64

## 7.6 附录六：支付流程详解

### 7.6.1 消费流程

![1615446646853](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446646853.png)

### 7.6.2 消费撤销流程

![1615446657825](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446657825.png)

### 7.6.3 消费退货流程

![1615446667412](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446667412.png)

## 7.7 附录七：返回码

| 返回码 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 94     | 没有通过人像验证，请次日后再试                               |
| 95     | 您没有通过人像验证                                           |
| 96     | 当前用户未开通人像照片认证                                   |
| a10    | 不合法的backend_token，或已过期（参见6.1.1获取backendToken章节，重新获取backend_token） |
| a20    | 不合法的frontend_token，或已过期（参见6.1.2获取frontToken章节，重新获取front_token） |
| a31    | 不合法的授权code，或已过期（参见5.3系统对接步骤章节，参见常见问题解答） |
| BC2126 | 缓存中不包含此商户appId                                      |
| BC2127 | 缓存中不包含此backendToken                                   |
| BC2133 | 获取商户信息异常                                             |
| N60005 | cdhdUsrId为空                                                |
| N62100 | 商户appId不能为空                                            |
| N62101 | backendToken不能为空                                         |
| N62102 | frontToken不能为空                                           |
| N62103 | scope不能为空                                                |
| N62104 | 签名字段值不能为空                                           |
| N62105 | responseType应为固定值code                                   |
| N62106 | grantType应为固定值authoriz                                  |
| N62107 | 随机字符串不能为空                                           |
| N62108 | 时间戳不能为空                                               |
| N62109 | accessToken不能为空                                          |
| N62110 | openId不能为空                                               |
| N62111 | url不能为空                                                  |
| N62112 | code不能为空                                                 |
| N62113 | contractCode不能为空                                         |
| N62114 | planId不能为空                                               |
| N62115 | contractId不能为空                                           |
| N62116 | content不能为空                                              |
| N62117 | secret不能为空                                               |
| N62118 | actionType不能为空                                           |
| N62119 | channelNo不能为空                                            |
| N62120 | region不能为空                                               |
| N62121 | result不能为空                                               |
| N62122 | bizOrderId不能为空                                           |
| N62123 | bizType不能为空                                              |
| N62124 | merchantId不能为空                                           |
| N62125 | notifyUrl不能为空                                            |
| N62126 | 缓存中不包含此商户appId                                      |
| N62127 | 缓存中不包含此backendToken                                   |
| N62128 | 接口名不能为空                                               |
| N62129 | relateId不能为空                                             |
| N62130 | trId不能为空                                                 |
| N62131 | 无权限访问此接口（请检查授权联登上送scope是否正确，咨询业务人员是否具有该接口调用权限） |
| N62132 | 不支持此访问域名（请检查申请对接云闪付开放平台的申请表中配置的域名与授权联登请求链接中的redirect_url是否一致） |
| N62133 | 获取商户信息异常                                             |
| N62134 | 退税号不能为空                                               |
| N62135 | 用户授权时，没有提供手机号                                   |
| N62136 | 用户没有手机号                                               |
| N62137 | 获取签约商户信息异常                                         |
| N62138 | 验证支付密码异常                                             |
| N62142 | 待更新的缓存key不能为空                                      |
| N62143 | 待更新的缓存expire不能为空                                   |
| S52131 | 无权限访问此接口（请检查授权联登上送scope是否正确，咨询业务人员是否具有该接口调用权限） |
| S52136 | 用户没有手机号                                               |

 

## 7.8 附录八：常见问题解答

| 序号               | 问题描述                                                     | 解决方案                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1.                 | 授权回调url不支持                                            | Ø 检查申请对接云闪付开放平台的申请表中配置的域名，与请求链接中的redirect_url回调地址的域名是否一致。 |
| 2.                 | upsdk is not defined                                         | Ø 检查是否引用upsdk.js。  Ø 检查依赖js（先于upsdk.js引用），zepto或jquery是否引用，zepto版本是否高于1.0，jquery版本是否高于1.4，建设使用最新版本。  Ø 检查upsdk是否全小写。 |
| 3.                 | 请求非法                                                     | Ø 检查时间戳是否获取正确，需获取东八区北京时间，精确到秒。  Ø 检查时间戳是否有效，有效期为5分钟。 |
| 4.                 | 支付控件报错                                                 | Ø 该错是全渠道接口返回错误，通过以下地址查询具体含义：  https://open.unionpay.com/ajweb/help/respCode/respCodeList |
| 5.                 | 请求报文解析错误                                             | Ø 检查请求报文是否符合规范，必须为http请求，标准json格式， UTF-8编码，对中文进行过处理。 |
| 6.                 | 请求token过期                                                | Ø 检查accessToken是否过期，有效期1小时。  Ø 检查backendToken frontToken 是否过期，有效期 2小时。  Ø 检查code是否过期， 有效期 5分钟。 |
| 7.                 | upsdk.config报错                                             | Ø 根据upsdk.error提示信息检查：  Ø 检查时间戳、随机字符串与签名因子是否一致。  Ø 检查fronttoken是否在有效期内，有效期为2小时。  Ø 检查上送的url是否在申请对接云闪付开放平台的申请表中配置的域名内。  Ø 检查上送的url是否为当前网页的url。  Ø 检查签名是否按照文档指引进行的签名 (url获取是否正确，签名串是否字典排序，签名url是否不包含#及其后面部分) 。  Ø 检查签名的因子是否正确，应包含appId、nonceStr、timestamp、url、frontToken。 |
| 8.                 | 不合法APPSECRET                                              | Ø 检查签名因子和上送的请求参数secret是否正确。               |
| 9.                 | appId 不合法                                                 | Ø 检查请求参数appId和提供的是否一致。                        |
| 10.                | 不合法scope                                                  | Ø 检查请求的scope权限是否匹配，咨询业务人员查看权限。        |
| 11.                | upsdk.config接口验签失败                                     | Ø 检查签名是否按照文档指引sha256算法签名。  Ø 检查签名串是否进行字典排序。  Ø 检查url是否通过window.location.href获取（签名的url不包含#及其后面部分）。 |
| 12.                | 解密失败                                                     | Ø 检查是否按照接入文档2.4安全规范章节中的对称密钥解密方法java版Demo参照解密。 |
| 13.                | 不合法code/不合法openId                                      | Ø code或者openId若是从前端传到后台，如果中间有+号，会转义成空格，+号可以替换成%2B解决。 |
| 14.                | 不合法的授权code，或已过期                                   | Ø 检查code中间是否有空格，或者backendToken是否已过期。ps 不建议两个系统用同一套参数，会导致backendToken互相被顶掉，如果是由一个系统获取、维护backendToken，可以复用一套参数。 |
| 15.                | The server has understood the request, but refused to implement  it. | Ø 检查调用的接口路径，user.mobile 容易误写成user_mobile。    |
| 16.                | 安卓图片不显示                                               | Ø 检查https网页图片是否用了http链接，安卓5.0版本以上WebView默认不支持同时加载Https和Http混合模式，建议图片服务器改成https链接。 |
| 17.                | 参数格式错误                                                 | Ø 检查请求参数名称，参数个数是否与文档一致。                 |
| 18.                | 页面缓存如何处理                                             | Ø 默认情况下WebView有缓存逻辑（iOS缓存时间相对较长），为了使修改的代码立即生效，对于前后端分离的开发者建议将加载文件后缀加上时间戳或者哈希值来改变文件路径，强制WebView加载最新文件。 |
| 19.                | 没有调用该接口的权限                                         | Ø 检查请求的scope是否正确，咨询业务人员是否具有该接口调用权限。 |
| 20.                | 调用接口返回系统异常,请稍后再试                              | Ø 检查上送接口内容是否采用json报文格式。                     |
| 21.                | 云闪付useragent                                              | Ø com.unionpay.chsp ；com.unionpay.mobilepay；updebug        |
| 22.                | upsdk验签怎么签名                                            | Ø appId、nonceStr、timestamp、url、frontToken字段字典排序成串后，用sha256方法计算签名值 |
| 23.                | 怎么关闭页面                                                 | Ø 使用upsdk关闭webview的接口                                 |
| 24.                | 用户点击暂不授权如何处理                                     | Ø 根据回调errmsg和errorcode处理，返回”用户不同意授权[user0001]”和user0001信息进行处理 |
| 25.                | 接口报文返回“请求非法”                                       | Ø 上送时间戳是否获取正确，如获取是服务器时间，检查是不是东八区北京时间，取秒，上送的时间戳有效期5分钟 |
| 26.                | Upsdk插件报”请求非法”                                        | Ø 检查页面地址是否在申请的安全域名内                         |
| 27.                | 联登报”授权回调url不支持”                                    | Ø 上送的redirectUri不在申请的联登域名内                      |
| 28.                | 左上角返回键是否有监听接口                                   | Ø 左上角是系统返回键，无接口监听                             |
| 29.                | 访问控制拒绝了你的请求                                       | Ø 请求频率不能超过1000次/分钟，否则会被加入黑名单，封禁24小时 |
| 30.                | 系统异常，请稍后再试                                         | Ø 后台接口均需为 post   json 请求。                          |
| 31.                | 客户端闪退                                                   | Ø url中包含中文字符，不同安卓系统和浏览器解析会存在兼容性问题，造成客户端闪退。解决方法：通常做法是中文转码后再拼入url，比如用base64  Ø 安卓10.0以上系统，建议云闪付版本更新到7.0以上 |
| 请求全渠道接口报错 |                                                              |                                                              |
| 32.                | 商户状态不准确                                               | Ø 联系签约收单机构查询原因。可能是商户号已注销，三个月无交易的商户号会自动注销 |
| 33.                | 接收不到后台通知                                             | Ø 检查通知接收后台是否活动，可自己POST一条消息检测。银联全渠道后台通知以标准的HTTP协议的POST方法向接入方上送的后台通知URL发送，超时时间为10秒。 |
| 34.                | 全渠道接口支持                                               | Ø https://open.unionpay.com/ajweb/help/respCode/respCodeList |
| 35.                | 未收到交易结果通知                                           | Ø 对于未收到交易结果的联机交易，商户应向银联全渠道支付平台发起交易状态查询。建议商户在发起消费类交易后，先等待交易结果推送，如未收到交易结果，5分钟后再发起交易状态查询。查询5次以上，仍获取不到明确状态的交易，后续可以间隔更长时间发起查询，最终结果以对账文件为准。 |

 

## 7.9 附录九：用户无感支付签解约流程

![1615446681086](C:\Users\zhipenghan\Documents\云闪付开放平台.assets\1615446681086.png)

 

## 7.10 附录十：红包/抽奖接入流程指引

“应用主管方在申请调用赠送红包/抽奖接口时，应填写《云闪付事业部营销业务接口调用信息登记表》，表格见附件。正常业务开展流程如下：

1、 接入方先通过UOSP提交工单申请营销活动业务参数。

如果接入方开展红包活动，服务目录：业务管理—云闪付营销活动支持—全额入账营销活动支持—积分业务预算授权码申请维护：若只调用赠送红包接口，则申请项目勾选积分业务（普通）；若调用抽奖接口（可赠送非定额红包），则勾选积分业务（消费送红包）。

如果接入方开展消费送券/激励金活动，服务目录：业务管理—云闪付营销活动支持—全额入账营销活动支持—立减、消费送现金业务预算码申请维护；同时申请项目应根据实际赠送奖励据实勾选。

2、 如果是申请测试活动，请在活动名称中标注测试，无需M8事由申请单。如果是生产活动，则需要在M8中申请事由申请单，相关预算授权码会在M8中反馈。申请流程审批通过后，如果是红包活动，通过流程接入方获得预算授权码和接入方代码；如果是抽奖活动，获得预算授权码。

3、 如接入方调用抽奖活动，在调用接口前，应联系云闪付事业部权益与营销团队李雪垠/胡源涛，根据实际要赠送的奖励类型配置抽奖活动，并获得抽奖活动ID和资格池编号UUID。

4、 在相关参数生成后，接入方需将营销费用打款到银联账户，备注预授权码。（如未备注联系服务经理，提交打款证明材料）。打款成功后，联系服务经理进行预算授权码或接入方账户充值。

5、 在充值成功后，接入方通过UOSP提交‘云闪付APP内容接入申请服务单’，同时上传《云闪付事业部营销业务接口调用信息登记表》。内容接入申请表中‘账户机构代码（接入方代码）’填写第一步中分配的接入方代码（接入方代码前面需加T00，对应接口上送的insAcctId字段的value值前面加T00，示例：T00T00000000888888），活动ID填写前述分配的活动ID和资格池编号UUID。”

备注: 1.红包 (机构账户)余额不足时，赠送返回成功应答，实际不会进行任何账户操作。

2.抽奖（红包/票券）余额不足时，返回"resp":" AM03051".3.接入方名称须小于40字节，20中文。
