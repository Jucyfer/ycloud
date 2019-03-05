

### 1. 接口概述

所有请求http请求头部设置`Content-Type:application/json;charset=utf-8;`

#### 1.1 接口请求校验

apikey在http头部放置`Authorization:you_api_key`

#### 1.2 接口权限校验失败时.

http_response:401
response:

```json
{
    "timestamp": 1545033511912,
    "status": 401,
    "error": "Unauthorized",
    "message": "INVALID_API_KEY",
    "path": "/v1/account/balance"
}

```

#### 1.3 接口响应结构说明

http 返回200说明请求正常到达服务器.

接口基础返回值:

名称|类型|描述
---|---|----
code|int|code=0时,请求成功,code!=0时,说明请求异常
msg|String|请求成功时"success",失败时,为失败的描述

其他返回值:
当业务参数不存在时,返回json中没有此字段,即当字段为null时,json序列化时忽略此字段.



### 2. SMS相关api 

#### 2.1 短信发送

`URL POST https://api.ycloud.com/v1/sms/send_messages`

`curl -H "Authorization:you_api_key" -H "Content-Type:application/json" -X POST --data '{"mobile":"+8615355498963","text":"你好,世界"}' www.baidu.com`

**必须参数**

参数|是否必须|类型|描述|示例
---|---|---|---|---
mobile|true|String|手机号,遵循E.164格式|+8615355498966
text|true|String|短信内容|你好,世界
uid|false|String|用户的业务id|1234567
callback_url|false|String|状态报告推送url|http://abcdaa.c.com/callBack
extend|false|String,内容必须是数字.|扩展号


**request示例**

```json
{
"mobile":"+8615355498963",
"text":"hello,world!"
}

```


**响应参数**

参数|是否必须|类型|描述|示例
---|---|---|---|---
data|false|Sms对象|短信对象,当提交成功时才有此字段

**Sms对象**

参数|是否必须|类型|描述|示例
---|---|---|---|---
region_code|true|String|国家地区码|CN
status|true|String|短信状态,详细见 附录|accepted
price|true|float|本次发送总共消耗金额|0.03
count|true|int|计费条数|1
mobile|true|String|手机号|+8615355498966
sid|true|String|唯一id.一般为int64数字转为字符串|3310228982
create_timestamp|true|timestamp|时间戳精确到毫秒|1541580135423

**response示例**
**提交成功:**

```json

{
    "code": 0,
    "data": {
        "status": "accepted",
        "price": 0.1582,
        "count": 2,
        "mobile": "+81486259891",
        "sid": "3321910440",
        "region_code": "JP",
        "create_timestamp": 1545040926268
    },
    "msg": "SUCCESS"
}
```

**提交失败**

```json

{
    "code": 2,
    "msg": "BAD_ARGUMENT The mobile is unverified"
}

```

#### 2.2状态报告推送

当收到推送时必须返回200,否则认为接收失败.后续将重试5次,1min 10min 60min 24h 72h

推送方式:

`POST  <callback_url>`

curl示例
`curl -H  -d'{"sid":"3310228982","status":"delivered","push_timestamp":1541580135423,"mobile":"+8615355498963"}' -X POST http://my.sms.com/call_back`

**推送参数**

参数|是否必须|类型|描述
---|--|--|---
sid|true|String|短信id
status|true|String|状态.
push_timestamp|true|timestamp|时间戳精确到毫秒
mobile|true|String|手机号
uid|false|String|
error_code|false|String
error_msg|false|String
 


#### 2.3短信记录获取

`GET https://api.ycloud.com/v1/sms/list`

请求频率限制:1次/1min

`curl -H "Authorization:you_api_key" -X GET https://api.ycloud.com/v1/sms/list`


**query**

参数|是否必须|类型|描述
---|---|---|---
page|true|int|页码从1开始
page_size|true|int|页面容量默认100,最大5000
from|true|timestamp|开始时间精确到毫秒
until|true|timestamp|结束时间精确到毫秒
sid|false|String|这个参数传入，其他值忽略

**响应参数**

参数|是否必须|类型|描述
---|----|----|----
count|true|int|数量
page_no|true|int|页码,从1开始
page_size|true|int|-
data|true|array\<Sms\>|Sms对象数组
msg|true|String|
code|true|int|返回状态


**Sms对象**

参数|是否必须|类型|描述|示例
---|---|---|---|---
region_code|true|String|国家地区码|CN
status|true|String|短信状态,详细见 附录|accepted
price|true|float|消耗金额|0.03
unit|true|String|price单位|dollars
count|true|int|计费条数|1
mobile|true|String|手机号|+8615355498966
sid|true|String|唯一id.一般为int64数字转为字符串|3310228982
text|true|String|短信内容
error_code|false|String|错误码,当短信发送不成功时会有.|YC:NO BALANCE
create_timestamp|true|timestamp|短信提交时间戳,精确到毫秒|1541580135423
status_timestamp|false| timestamp|状态报告时间戳,精确到毫秒| 1541580135423
push_timestamp |false| timestamp|推送用户时间戳,精确到毫秒| 1541580135423

**response示例1**

```json

{
    "code": 0,
    "data": [
        {
            "status": "delivering",
            "price": 0.0087,
            "count": 1,
            "mobile": "15158567221",
            "sid": "3321910100",
            "text": "demoData??????",
            "region_code": "CN",
            "create_timestamp": 1545038673722,
            "push_timestamp": 0
        }
    ],
    "msg": "SUCCESS",
    "count": 0,
    "page_no": 1,
    "page_size": 1
}


```
**response示例2没有数据时**

```json

{
    "code": 0,
    "data": [],
    "msg": "SUCCESS",
    "count": 0,
    "page_no": 1,
    "page_size": 1
}


```


#### 2.4上行推送

当收到推送时必须返回200,否则认为接收失败.后续将重试5次,1min 10min 60min 24h 72h


`POST <inbound_callback_url>`

`curl -d '{"id":"a1s1we1111","mobile":"+8615355498966","text":"谢谢","inbound_timestamp": 1541580135423}' -X POST http://my.sms.com/inbound_callback`


参数|是否必须|类型|描述
---|---|---|---
id|true|String|唯一id
mobile|true|String|回复手机号
text|true|String|回复的短信内容
inbound_timestamp|true| timestamp |时间戳



### 3.账户相关接口
#### 3.1余额查询

`GET https://api.ycloud.com/v1/account/balance`

`CURL -H "Authorization:you_api_key" -X GET https://api.ycloud.com/v1/account/balance`

**响应参数**

参数|是否必须|类型|描述
---|---|---|---
count|true|int|数量
msg|true|String|
balance|true|Balance对象|-


**account对象**

参数|是否必须|类型|描述|示例
---|---|---|---|---
currency|true|String|币种,目前 USD,CNY|USD
amount|true|float|余额,精确到小数点后五位|3.44


**响应示例1**

```json

{
    "code": 0,
    "data": {
        "currency": "USD",
        "amount": 10000010190.07658
    },
    "msg": "SUCCESS"
}

```

**响应示例2**

```json

{
	"amount":10000010190.07658,
	"currency":"USD"
}

```

### 附录

#### 附录1.code

code|描述|detail
---|---|-----
-2|NO_BASE_PRICE
-1|INVALID_API_KEY
0|SUCCESS
1|ARGUMENT_MISSING
2|BAD_ARGUMENT
3|NO_BALANCE


#### 附录1.status 

status|描述|
---|---
unaccepted|未接受
accepted|接口提交成功
failed|其它原因未提交供应商。我们对应的有mc余额不足或无效通道。 对应错误码： YC:XXXXXX
sent|通道已分配,状态报告待返回
delivered|已经下发
undelivered|供应商返回失败，对应错误码：OP:XXXXXXX （包含mc提交失败返回）


#### 附录2.error_code

前缀|描述
----|----
YC:|"YC:"开头的,非运营商
OP:|"OP:"开头运营商返回




**非运营商**

error_code|描述
-----|-------
YC:13|营销短信暂停发送
YC:20|不支持的国家地区
YC:MOBILE BLACK	|号码在账号黑名单中，不扣费
YC:NO BALANCE|余额不足扣费失败
YC:NOOP|无效通道，不扣费
YC:NOSEND|无效通道，不扣费



**运营商相关**
OP:+运营商的返回值
