# CNNS H5小程序开放平台
* [概述](#概述)
* [开放形式](##开放形式)
* [开放功能](##开放功能)
   * [对接方式](###对接方式)
   * [帐号对接](###帐号对接)
   * [数字货币收款](###数字货币收款)
   * [数字货币付款](###数字货币付款)

## 概述
CNNS H5小程序开放平台是对外提供第三方H5 APP对接进CNNS 生态内各个APP能力的一个开放应用平台。本文档为接入的框架指南，详细的接口与SDK说明，请查看相关文档。

[CNNS H5小程序开放平台(JS-SDK接入文档)](https://github.com/NetGodFather/cnns-minapp-open/blob/master/README-JS-SDK.md)

[CNNS H5小程序开放平台服务端对接文档](https://github.com/NetGodFather/cnns-minapp-open/blob/master/README-SERVER.md)
## 开放形式
平台当前开放的形式，是通过将第三方使用H5开发的应用，内嵌到 CNNS 生态下的 APP 中，并通过APP以及HTTP服务，为应用开发者提供各种额外的功能。
> 所有开放的功能，都需要提前申请才可以使用。

## 开放功能
CNNS H5 小程序平台，当前对外开放的核心功能有：
 1. 帐号对接：可以直接使用APP的账号体系，免去用户注册、登录的麻烦；
 2. 数字货币收款：无须对接区块链，即可实现向用户收取数字货币的能力；
 3. 数字货币付款：无须对接区块链，即可实现向用户支付数字货币的能力；
 4. 调整屏幕方向：可以自由的调增屏幕方向为 横屏或者竖屏；

### 对接方式
除了调整屏幕方向以及一些简单的信息获取，其他的功能都需要前端以及服务端调用协同一起完成。以下就各个功能的实现流程做一个简单介绍：
### 帐号对接
帐号对接，是指应用直接使用 APP 的账号体系，免去用户注册和登录的麻烦。在平台内部的所有帐号已经打通，应用无须关注具体是来自那个APP的用户。
**流程如下：**
1. 首先应用需要通过 JS-SDK 的 isInApp() 方法，检测当前是否在支持的开放平台的APP内。如果确认在，进入第二步；
2. 调用 cwLogin 方法，将 app_id 和 登录成功后的回调函数传入。此处SDK 也会检测，如果不在APP 内，调用会返回 false；
3. 如果回调登录成功，将获得 code 参数，将此参数通过开发者自己的服务器，调用 open.auth.token 方法，获取当前用户的 open_id 及授权( auth_token )；
   1. 如果 open_id 是新用户，可以通过调用 open.user.info 获取用户的基本信息；
   2. 返回的 auth_token 可以在 expires_in 指定的时间之前使用，过期后需要通过接口返回的 refresh_token 再次调用此接口，重新活动。refresh_token 30天过期，如果过期，则需要重新授权；
4. 完成以上步骤后，则表示用户登录成功，应用需要保存当前用户登录状态，为用户提供服务；

### 数字货币收款
通过数字货币收款功能，应用可以向用户发起收款，等用户确认后。该数字货币将转到应用的帐户（所收到数字货币的相关结算方式，请单独和平台沟通）。

**流程如下：**
1. 首先应用应该告知用户所提供的服务内容和价格，待用户确认后再发起收款；
2. 发起授权，需要先通过调用服务端的接口 open.pay.inorder.create 创建收款订单；
3. 创建收款订单成功后，可以获得收款订单编号 out_pay_id ；
4. 通过 JS-SDK 调用 cwPay 方法，让 APP 唤起确认支付的窗口，等待用户确认付款；
5. 用户确认付款，并且付款成功后。可以通过回调收到付款订单信息中的 status 属性确认是否支付成功；
6. 强烈建议在收到回调后，再次通过服务端的接口 open.pay.in_order.info 获取订单信息，确认是否真的支付成功；

### 数字货币付款
数字货币付款是直接给用户转账数字货币。该接口为高级接口，需要额外申请。付款因为是通过应用向用户转账，所以无需用户确认。
**流程如下：**
1. 应用确认需要转账用户和币种以及数量；
2. 调用 open.pay.out_order.create 创建付款订单，该创建过程是同步的，即调用立即会返回是否付款成功；
3. 如果调用出现异常，可以通过 open.pay.out_order.info 查询，确认是否付款成功，避免重复付款；