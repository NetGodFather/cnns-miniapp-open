# CNNS H5小程序开放平台

* [概述](#概述)
  * [基础通讯](##基础通讯)
	  * [请求方式](###请求方式)
	  * [请求参数](####请求参数)
       * [返回值](####返回值)
 * [接口](#接口)
	 * [授权登录](##授权登录)
		 * [授权码获取/刷新](###授权码获取/刷新)
	 * [用户](##用户)
		 * [获取用户信息](###获取用户信息)
		 * [获取帐户余额](###获取帐户余额)
	 * [收款](##收款)
		 * [创建收款订单](###创建收款订单)
		 * [查询收款订单](###查询收款订单)
	 * [付款](##付款)
		 * [创建付款订单](###创建付款订单)
		 * [查询付款订单](###查询付款订单)
 * [标准数据结构](#标准数据结构)
	 * [用户信息 (user)](##用户信息 (user))
	 * [收款订单信息 (in_order)](##收款订单信息 (in_order))
	 * [付款订单信息 (out_order)](##付款订单信息 (out_order))
 * [状态码/错误码](#状态码/错误码)
	 * [状态码表](##状态码表)
	 * [错误码表](##错误码表)

## 概述
CNNS H5小程序开放平台是对外提供第三方H5 APP对接进CNNS 生态内各个APP能力的一个开放应用平台。

## 基础通讯
为了方便第三方获取用户信息，执行一些相关操作。开放平台通过 openapi.bishijie.com 对外提供的服务端的接口调用能力。

### 请求方式
所有服务端接口使用 HTTPS 协议，请求地址为https://openapi.bishijie.com 。请求分 GET 和 POST 两种方式，一般信息获取类都是 GET，有安全风险或涉及数据写入的，使用POST。
两端的所有请求和返回值，都附带有 sign 签名参数。sign 是使用RSA密钥，通过 RSA2 签名算法进行签名。请求参数的签名使用应用方私钥进行签名，返回值的验签名，使用平台方提供的公钥进行验签（每个APP对应的公钥都不相同）。

### 请求参数
请求参数分接口基础参数与业务参数两类，基础参数是所有请求都需要有的参数，业务参数根据不同的接口，会各有不同。业务参数会通过 JSON 打包成字符串，放到 biz_content 中传输。

请求的参数如下：

|参数|说明|
|-|-|
|app_id|应用方的应用编号，由平台方提供|
|method|请求方法，如：open.auth.token|
|format|数据格式，固定为 json|
|charest|编码，固定为 utf-8|
|sign_type|签名方式，固定为 RSA2|
|timestamp|时间戳，单位毫秒|
|version|接口版本，当前为 v1.0|
|biz_content|通过 json 打包的业务请求参数。|
|auth_token|用户登录授权的 auth_token ，所有涉及用户的接口，此参数都必填|
|sign|使用应用端私钥通过 RSA2 对以上请求信息进行签名得到的字符串|

### 返回值

请求的返回值，包含请求返回码(错误码)、返回信息和返回值三部分组成。其中返回值是业务请求执行返回的业务数据，通过 format 指定的方式进行打包（当前仅支持 json )。完整返回信息如下：

|参数名|说明|
|-|-|
|code|状态码，接口请求的状态（与业务是否执行成功达到预期无关）|
|message|请求装填描述信息，code 的文字描述|
|biz_response|业务请求返回数据（ json 格式）
|timestamp|当前返回的时间戳（可以避免中间节点，篡改成旧的返回信息从而产生可能的安全风险）|
|sign|返回值的签名，可以通过平台提供的服务端公钥进行验签。|

> 对于在业务请求中发生的错误，会放到 biz_response 里边。通过 error_code , error_message
> 返回错误码和错误描述。

# 接口

## 授权登录

### 授权码获取/刷新

**接口方法：** open.auth.token

**请求方式：** POST

**接口说明：** 通过APP+H5 前端登录获取的验证码( code )，获取当前用户授权登录的相关信息。每调用一次，获得的 access_token 会延长一定的有效期，确保通信安全；

**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|code||string(40)|通过 H5+APP 请求登录授权获得的当前用户登录 code|
|refresh_token||string(40)|通过之前调用该接口获取的 refresh_toke 获取最新的 auth_token|

> code 、refresh_toke 二选一，不可两个都提供

**返回值：**

|参数|数据类型|说明|
|--|--|--|
|open_id|string()|开放平台针对用户的 open_id|
|auth_token|string()|用于各个业务请求时候的用户授权身份确认|
|expires_in|int|auth_token 将在多长时间之内过期，需要在过期之前调用 refresh_token 重新获取；|
|refresh_token|string()|可以用于刷新 auth_token 过期时间的token，refresh_token 将在 30天后过去，30天后，需要重新授权登录|

## 用户

### 获取用户信息

**接口方法：** open.user.info
**请求方式：** GET
**接口说明：** 通过 open_id 获取用户资料
**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|open_id|Yes|string(40)|用户open_id（必须传相匹配的 auth_token 参数）|

**返回值：**
> 与 user 数据结构完全一样。

### 获取帐户余额
**接口方法：** open.user.balance

**请求方式：** GET

**接口说明：** 通过 open_id 获取用户持有某个币种的余额熟练；

**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|open_id|Yes|string(40)|用户open_id（必须传相匹配的 auth_token 参数）|
|coin_ids|Yes|string|币种编号可以通过提供逗号分割的多个币种编号获取多个币种的余额（最多可指定10个）|

**返回值：**

|参数|数据类型|说明|
|-|-|-|
|open_id|string|开放平台针对用户的 open_id|
|balances|map|币种ID作为KEY，余额作为值组成的 map，其中值为 float 累心，示例如：{cnns:12031.10000, btc:0.021200}|

## 收款

### 创建收款订单

**接口方法：** open.pay.inorder.create

**请求方式：** POST

**接口说明：** app 创建向用户收款的订单，如果创建成功，将返回创建的订单对象。

**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|open_id|Yes|string(40)|下单的用户open_id（必须传相匹配的 auth_token 参数）|
|in_order_id|Yes|string(32)|应用端的关联的订单编号(32位字符串)|
|title|Yes|string(50)|所购买商品的描述（100个字符以内）|
|coin_id|Yes|string(20)|所需要扣款的币种编号|
|amount|Yes|float|需要支付的金额|
|state|No|string(100)|额外附加的自定义参数（100个字符以内)|

**返回值：**
> 与 in_order 数据结构完全一样。

### 查询收款订单

**接口方法：** open.pay.in_order.info

**请求方式：** GET

**接口说明：** 查询某个收款订单的信息，一般用于对账或者检查某个订单是否支付成功。

**请求参数：**
|参数|是否必填|数据类型|说明|
|-|-|-|-|
|in_order_id||string(32)|应用端关联的订单编号
|in_pay_id||int|有创建订单接口返回的收款订单编号（建议优先使用）|
> in_order_id 和 in_pay_id 二选一，必填一个

**返回值：**

> 与 in_order 数据结构完全一样。

## 付款

### 创建付款订单

**接口方法：** open.pay.out_order.create

**请求方式：** POST

**接口说明：** 应用方向指定的某个用户支付一定数额的数字货币。

**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|open_id|Yes|string(40)|下单的用户open_id（必须传相匹配的 auth_token 参数）|
|out_order_id|Yes|string(32)|应用端的关联的订单编号(32位字符串)|
|coin_id|Yes|string(20)|所需要扣款的币种编号|
|title|Yes|string(50)|付款的原因（50个字符以内，需要展示给用户的）|
|amount|Yes|float|需要支付的金额|
|state|No|string(100)|额外附加的自定义参数（100个字符以内)|

**返回值：**

> 与 out_order 数据结构完全一样。

### 查询付款订单

**接口方法：** open.pay.out_order.info

**请求方式：** GET

**接口说明：** 查询指定的付款订单

**请求参数：**

|参数|是否必填|数据类型|说明|
|-|-|-|-|
|out_order_id|string(32)|应用端关联的订单编号|
|out_pay_id|int|有创建订单接口返回的收款订单编号（建议优先使用）|
> out_order_id 和 out_pay_id 二选一，必填一个

**返回值：**
> 与 out_order 数据结构完全一样。

# 标准数据结构

1.  标准数据结构是指一些结构化数据的标准输出格式（如订单信息、用户信息）。这种格式在所有接口的时候，结构都一样。
2.  接口提供方和接入方都可以考虑将这类数据的生产和输出处理标准化，简化开发难度。
3.  如果接口中返回的是标准的数据结构，我们将不会详细列出，而只会说明数据结构的名称；
4.  相同的数据，因为需求度的不同里边的数据项目需求不同，可能会分成多个数据结构；

## 用户信息 (user)
|属性|名称|类型|说明|
|-|-|-|-|
|open_id|用户open_id编号|string||
|nickname|用户昵称|string||
|headimgurl|用户头像|string||

## 收款订单信息 (in_order)
|属性|名称|类型|说明|
|-|-|-|-|
|in_pay_id|系统订单号|int|钱包系统记录的收款订单的订单编号（全系统唯一）|
|app_id|应用编号|string|创建订单的应用编号（应用只能查询自己的订单）|
|in_order_id|应用订单编号|string(32)|应用端创建订单的时候，提供的订单号|
|title|订单标题|string(50)|订单标题，需要展示给用户的|
|open_id|下单用户编号|string(20)|下单用户的 open_id 编号|
|coin_id|币种编号|string(20)|下单支付的币种编号|
|amount|支付数量|float|下单支付的数量|
|state|附件参数|string(100)|下单时候所提供的附加参数|
|status|订单状态|int|0: 未支付；1：已支付成功；-1：支付失败|
|fail_code|失败编码|string|用于代码中识别失败原因（如果支付失败，未失败为空）|
|fail_reason|失败原因|string|用于方便调试时理解失败原因（如果支付失败，未失败为空）|
|create_time|创建时间|long|创建订单的时间，精确到毫秒|
|pay_time|支付时间|long|最后支付成功的时间戳，精确到毫秒|

## 付款订单信息 (out_order)
|属性|名称|类型|说明|
|-|-|-|-|
|out_pay_id|系统订单号|int|钱包系统记录的收款订单的订单编号（全系统唯一）|
|app_id|应用编号|string|创建订单的应用编号（应用只能查询自己的订单）|
|out_order_id|应用订单编号|string(32)|应用端创建订单的时候，提供的订单号|
|title|订单标题|string(50)|订单标题，需要展示给用户的|
|open_id|下单用户编号|string(20)|下单用户的 open_id 编号|
|coin_id|币种编号|string(20)|下单支付的币种编号|
|amount|支付数量|float|下单支付的数量|
|state|附件参数|string(100)|下单时候所提供的附加参数|
|status|订单状态|int|0: 未支付；1：已支付成功；-1：支付失败|
|create_time|创建时间|long|创建订单的时间，精确到毫秒|
|pay_time|支付时间|long|最后支付成功的时间戳，精确到毫秒|

  

# 状态码/错误码

1.  状态码是指接口交互层面的请求状态反馈，会直接在返回值中提现。
2.  错误码是指业务执行中发生的错误，会打包在返回值的 response 中。
3.  状态码为3位，会参考 HTTP 协议错误码，200 被认为是成功调用，4XX 表示接口不存在或者不可用等方面的错误，5XX 表示系统异常方面的错误；
4.  错误码为4位，前两位是业务模块，后两位是错误序号；

## 状态码表


|状态码|说明|
|-|-|
200|请求成功
403|应用无权调用此接口
404|请求接口不存在
500|系统异常
501|应用编号无效或者已被禁用
502|签名验证失败
503|请求参数不完整

## 错误码表
错误码|错误说明|
|-|-|
|**授权部分**|
|1001|授权 code 无效|
|1002|refresh_token无效或已过期|
|1003|auth_token 无效或已过期|
|用户相关|
|1100|open_id 无效|
|1101|open_id 与 app_id 不匹配|
|1102|open_id 与 auth_token 不匹配|
|**收款订单部分**|
|1200|未设置应用关联帐号
|1201|收款标题设置无效
|1202|收款币种设置无效
|1203|收款数额设置无效
|1204|收款附加信息设置无效
|1210|收款订单编号重复
|1220|收款订单创建失败
|1250|收款订单不存在
|**付款订单部分**|
|1300|未设置应用关联帐号|
|1301|付款标题设置无效|
|1302|付款币种设置无效|
|1303|付款数额设置无效|
|1304|付款附加信息设置无效|
|1310|付款订单编号重复|
|1320|帐户余额不足付款失败|
|1321|付款数量超过限额|
|1350|付款订单不存在|
