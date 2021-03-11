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

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.gif)

## 4.3 系统对接步骤

**第一步：请求进入云闪付用户联登授权页，获取****CODE**

**云闪付****APP****外的接入方****APP**，使用云闪付SDK相关接口直接获得code。调用方式参考SDK文档及示例代码。

 

**云闪付****APP****内的接入方****H5**，跳转到此页面，请求获取code

https://open.95516.com/s/open/html/oauth.html?appId=APPID&redirectUri=REDIRECTURI&responseType=code&scope=SCOPE&state=STATE

 

**云闪付****APP****外**（如在微信公众号里）**的接入方****H5**，跳转到此页面，用户登录后获得code

https://open.95516.com/s/open/auth/outApp/html/outLogin.html?appId=APPID&redirectUri=REDIRECTURI&responseType=code&scope=SCOPE&state=STATE

 

云闪付APP外的SCOPE仅可使用upapi_login（同upapi_base）和upapi_user（同upapi_auth）。

 

强烈建议：跳转回调redirectUri使用https链接来确保授权code的安全性

请求参数说明：

| **序号** | **参数名**   | **是否必填** | **备注**                                                     |
| -------- | ------------ | ------------ | ------------------------------------------------------------ |
| 1.       | appId        | 是           | 接入方的唯一标识                                             |
| 2.       | redirectUri  | 是           | 授权后重定向的回调地址，请使用urlencode对链接进行处理（ios无需进行urlencode）。 |
| 3.       | responseType | 是           | 返回类型，请填写code                                         |
| 4.       | scope        | 是           | 应用授权作用域，参见4.1使用场景                              |
| 5.       | state        | 否           | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节 |
| 6.       | planId       | 否           | scope 为 upapi_contract  时上送。接入方定义的签约协议模板id，用于在签约页面中展示签约内容 |

 

登录之后页面将跳转至 redirectUri?code=CODE&state=STATE（成功获取code时）

或 redirectUri?state=STATE&errmsg=XXXXYYYY&errorcode=xxx （失败时）

注意： 

Ø 返回code是通过encodeURIComponent经过URL编码的，接入方需要通过函数decodeURIComponent解码去获取code。

 

**第二步：调用云闪付开放平台接口，通过****CODE****获取****accessToken****和****openId**

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

