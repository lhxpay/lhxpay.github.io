---
layout: default
title: 联合支付对接说明文档
---


## 充值接口对接

### 充值接口参数说明

|参数|必须|描述|
|---|---|---|
|amount|是|订单金额
|appKey|是|商户用户接口调用的Key信息，如果不清楚请联系客服
|callbackUrl|是|充值成功后的回调接口，请求部分请参考回调参数说明
|channelId|是|支付通道ID，整数类型，请参考通道ID表
|mrn|是|商户平台订单号，字符串，最长可接纳64个字符
|returnUrl|是|客人充值完成后用于返回的网址
|sign|是|请求信息的签名信息，请参考充值接口签名说明
|payer|否|充值者的账户实名信息，注意，该参数为可选，不参与签名。如果有中文，请使用UTF-8编码
|sourceIp|否|充值者IP地址，如果为空，则默认是订单提交方的IP地址，该参数不参与签名。

### 充值接口签名说明

请求数据签名格式

`amount=金额&appKey=Key&callbackUrl=回调地址&channelId=通道ID&mrn=平台单号&returnUrl=返回地址&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

### 页面对接模式

页面对接模式即是将所有的充值接口参数按照`application/x-www-form-urlencoded`表单使用UTF-8编码以POST方式提交到相关的网关API `/api/Pay`，提交以后网页的控制权交与相关充值平台。

### WebAPI 对接模式

将所有的充值接口参数按照`application/json`以JSON对象的形式用POST方式提交到相关的网关API `/api/Pay/Create`，该接口将返回以下信息（JSON对象）

`{
    success: true/false,
    errorMsg: 错误信息,
    payUrl: 支付页面跳转地址
}`

其中，支付页面跳转地址仅包含URL中的Path和QueryString部分，没有Protocol和Host，比如/api/Pay?id=some-id&channel=SomeChannel，商户需要根据当前的相关对接网关地址拼接最后的跳转地址，跳转后，充值控制权交与相关充值平台。

### 充值回调参数说明

|参数|必须|描述|
|---|---|---|
|amount|是|客人实际支付金额，精确到分|
|appKey|是|商户用户接口调用的Key信息，如果不清楚请联系客服|
|channel|是|支付通道ID，整数类型，请参考通道ID表|
|mrn|是|商户平台订单号|
|originAmount|是|客人充值时提交的金额|
|sign|是|请求信息的签名信息，请参考回调签名说明|

### 回调签名说明

请求数据签名格式

`amount=实际金额&appKey=商户Key&channel=通道ID&mrn=平台单号&originAmount=请求金额&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

### 充值回调说明

平台将所有的回调参数按照`application/x-www-form-urlencoded`以表单的形式用POST方式提交到充值时所留下的callbackUrl参数所指定的地址中，IP白名单请联系客服，回调成功时，商户应返回HTTP Status 200并以`success`字样作为响应内容。

### 充值查单接口说明

HTTP WebAPI 请求

`GET /api/Payments/Query?appKey=商户Key&mrn=平台单号&sign=签名&callback=再次回调(true/false)`

签名格式

`appKey=商户Key&mrn=平台单号&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

JSON 响应内容

`{error: "success", "result": {Mrn: "平台单号", status: 0/1/2}}`

status 为0时表示等待客人支付，1表示支付成功，2表示支付失败。如果返回的status是1且callback是true，那么平台会再次回调callbackUrl，请参考充值回调说明。

### 充值通道表

|ID|名称|
|---|---|
|1|微信|
|2|支付宝|
|3|银行卡|
|4|支付宝红包|
|5|云闪付|
|6|支付宝转银行卡|
|7|支付宝wap|
|8|微信wap|
|9|支付宝跳转|
|10|微信跳转|
|11|微信手机转账 |
|12|支付宝转卡飞行模式|
|13|微信赞赏码|
|14|支付宝固码|
|15|支付宝群红包|

## 下发接口

### 下发接口参数说明

|参数|必须|描述|
|---|---|---|
|amount|是|订单金额，必须是整数，单位：元 |
|appKey|是|商户用户接口调用的Key信息，如果不清楚请联系客服|
|bankName|是|银行名称|
|accountName|是|银行账户名|
|accountNumber|是|银行卡卡号|
|mrn|是|商户平台订单号，字符串，最长可接纳64个字符|
|callbackUrl|是|出款成功后的回调接口，请求部分请参考回调参数说明|
|sign|是|请求信息的签名信息，请参考充值接口签名说明|

### 下发签名说明

请求数据签名格式

`accountName=账户名&accountNumber=卡号&amount=金额&appKey=Key&bankName=银行名称&callbackUrl=回调地址&mrn=平台单号&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

### 下发WebAPI说明

将所有的下发接口参数按照`application/json`以JSON对象的形式用POST方式提交到相关的网关API `/api/Payouts/Create`，该接口将返回以下信息（JSON对象）

`{
    error: "success",
    errorMessage: "错误信息"
}`

当error为success时表示下发请求成功，请等待回调。否则，请参考如下错误表

|错误代码|描述|
|---|---|
|invalid_data|无效的请求数据，请检查您的JSON数据和请求参数|
|sign_error|签名错误|
|mrn_duplication|平台单号冲突|
|insufficient_balance|商户余额不足|
|object_not_found|无效的商户账户|

### 下发回调参数说明

|参数|必须|描述|
|---|---|---|
|amount|是|下发金额，单位：元|
|appKey|是|商户用户接口调用的Key信息，如果不清楚请联系客服|
|mrn|是|商户平台订单号|
|sign|是|请求信息的签名信息，请参考回调签名说明|

### 下发回调签名说明

请求数据签名格式

`amount=实际金额&appKey=商户Key&mrn=平台单号&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

### 下发回调说明

平台将所有的回调参数按照`application/x-www-form-urlencoded`以表单的形式用POST方式提交到充值时所留下的callbackUrl参数所指定的地址中，IP白名单请联系客服，回调成功时，商户应返回HTTP Status 200并以`success`字样作为响应内容。

### 下发查单接口说明

HTTP WebAPI 请求

`GET /api/Payouts/Query?appKey=商户Key&mrn=平台单号&sign=签名&callback=再次回调(true/false)`

签名格式

`appKey=商户Key&mrn=平台单号&key=商户密钥`

>其中**商户密钥**为商户接口的密钥信息，如果不清楚请联系客服

签名为以上数据格式的MD5哈希以后32个字节的HEX小写字符串

JSON 响应内容

`{error: "success", "result": {Mrn: "平台单号", status: 0/1/2}}`

status 为0时表示等待下发，1表示下发成功，2表示下发失败。如果返回的status是1且callback是true，那么平台会再次回调callbackUrl，请参考下发回调说明。

## 示例代码

1. [PHP](https://github.com/lhxpay/phpsample/)
2. [Java](https://github.com/lhxpay/springbootsample/)
