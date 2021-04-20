# dmpay 商户接口文档
请先在 http://www.dmlabs.ltd 上注册账号，绑定谷歌验证码，创建App，获取appId和appScret

## 接口调用

### 请求
1. 接口地址: https://mch.dmlabs.ltd
2. 参数使用request的body传递，以json形式

### 返回值
1. 无特殊说明下所有返回值均为JSON，格式为{resultCode:”返回结果编码”,resultDesc:”返回结果文本描述”,resultData:{返回结果数据}}。resultData为对象类型，无结果情况下返回空

### 加/解密签名
1. 组成签名字符串
<br>所有参数key，除sign外，按照ASCII从小到大排序，用&连接成字符串，在最后拼上&appId=yourAppId
2. 使用SHA256加密获取签名
<br>将拼好的字符串再加上appScret，使用SHA256加密后获取签名
3. 举例
<br>以创建订单为例，参数

## 接口清单

### 创建付款订单
商户将商品描述、订单金额等信息提交给DMPay，由DMPay创建订单返回给商户，商户将返回值中的H5页面展示给用户即可。 用户可复制地址或扫描页面中的付款地址二维码使用钱包或交易所付款

<br>请求URL：/orders/create
<br>请求方式：post
<br>请求报文

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
appId|appId|String|32|否|由https://www.dmlabs.ltd中创建app可得
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
description|订单描述|String|32|否|将会在订单页面展示给用户看
notifyUrl|订单回调地址|String|128|否|订单回调地址，DMPay将订单的状态变化主动推送给业务系统
coinCode|付款币种|String|16|否|目前支持BTC/ETH/TRX/USDT，其中USDT支持TRC20/ERC20
amount|币种数量|String|16|否|用户需要支付的币种数量，支持6位小数
nonce|随机数|String|8|否|随机数
timestamp|商户订单时间|String|8|否|自1970年1月1日起至今的毫秒数
sign|签名字符串|String||否|参见 加/解密签名

<br>响应报文
参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
transactionId|订单id|String|32|否|由DMPay生成
orderUrl|支付页面链接|String|32|否|用户点击 我已支付时触发javascript函数：dmpaid()，如业务系统是原生APP，可拦截此方法，如果是H5页面，可以在父页面创建回调函数：dmpaidCallback(mchOrderCode)

### 创建提币订单
商户可通过此接口给用户提币。

<br>请求URL：/payment/create
<br>请求方式：post
<br>请求报文

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
appId|appId|String|32|否|由https://www.dmlabs.ltd中创建app可得
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
notifyUrl|订单回调地址|String|128|否|订单回调地址，DMPay将订单的状态变化主动推送给业务系统
coinCode|提币币种|String|16|否|目前支持BTC/ETH/TRX/USDT，其中USDT支持TRC20/ERC20
amount|币种数量|String|16|否|用户需要支付的币种数量，支持6位小数
toAddress|提币地址|String|16|否|提币目标地址
nonce|随机数|String|8|否|随机数
timestamp|商户订单时间|String|8|否|自1970年1月1日起至今的毫秒数
sign|签名字符串|String||否|参见 加/解密签名

<br>响应报文
参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
paymentId|提币订单id|String|32|否|由DMPay生成


## 回调接口

### 付款订单状态推送
DMPay检测到付款订单的状态变化时，会主动将信息推送至创建订单时指定的回调地址(notifyUrl)，且回调地址必须是https地址。注意订单会多次推送，请做好幂等处理。

<br>推送报文

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
transactionId|订单id|String|32|否|由DMPay生成
state|状态|String|32|否|NOT_PAY：待支付，PART_PAY：部分支付,SUCCESS：已支付，CLOSED：已关闭
sign|签名字符串|String||否|参见 加/解密签名

<br>业务系统处理成功后必须返回成功标识给DMPay，参数如下

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
status|处理状态|boolean|8|否|true：处理成功，false：处理失败


### 提币订单状态推送
DMPay检测到提币订单的状态变化时，会主动将信息推送至创建订单时指定的回调地址(notifyUrl)，且回调地址必须是https地址。注意订单会多次推送，请做好幂等处理。

<br>推送报文

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
paymentId|提币订单id|String|32|否|由DMPay生成
state|状态|String|32|否|NOT_PAY：待支付，NOT_AUDIT：待审核,READY：已审核待上链,NOT_CONFIRM：待链上确认,SUCCESS：已支付，CLOSED：已关闭
sign|签名字符串|String||否|参见 加/解密签名

<br>业务系统处理成功后必须返回成功标识给DMPay，参数如下

参数代码|参数名称|参数类型|参数(最大)长度|可空|说明
-|:-|:-|:-|:-|:-:
mchOrderCode|业务系统订单编号|String|32|否|业务系统提供
status|处理状态|boolean|8|否|true：处理成功，false：处理失败







