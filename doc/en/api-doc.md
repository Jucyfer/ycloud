

### 1. Overview

Your need to include following in your request headers:`Content-Type:application/json;charset=utf-8;`

#### 1.1 Authentication

With each API call, you will need to include your access key to authenticate yourself.`Authorization:you_api_key`

#### 1.2 Unverified

http_response:401

Response:

```json
{
    "timestamp": 1545033511912,
    "status": 401,
    "error": "Unauthorized",
    "message": "INVALID_API_KEY",
    "path": "/v1/account/balance"
}

```

#### 1.3 Request structure

Your platform should respond with a 200 HTTP header. 

Basic reponse value:

Name|Type|Description
---|---|---
code|int|When code=0, successfully requsted.When code!=0,exceptions that something goes wrong
msg|String|When successfully requsted,it will show "success". Failed description when request failed.

Other returned value:
When the parameter is not found,it will not be included in the JSON returned values, meaning the filed is ignored when JSON serialization.  



### 2. SMS API

#### 2.1 Send a message

`URL POST https://api.ycloud.com/v1/sms/send_messages`

`curl -H "Authorization:you_api_key" -H "Content-Type:application/json" -X POST --data '{"mobile":"+8615355498963","text":"Hello,world"}' www.baidu.com`

**Required parameters**

Paremeters|Required|Type|Description|Example
---|---|---|---|---
mobile|true|String|Mobile number with E.164 format|+8615355498966
text|true|String|SMS body|Hello world
uid|false|String|Unique id|1234567
callback_url|false|String|Delivery report push url|http://abcdaa.c.com/callBack
extend|false|String,numbers only.|Extension


**Request Example**

```json
{
"mobile":"+8615355498963",
"text":"hello,world!"
}

```


**Response parameters**

Paremeters|Required|Type|Description|Example
---|---|---|---|---
data|false|The message object|The message object occurs when the request is successfully created.

**The message object**

Paremeters|Required|Type|Description|Example
---|---|---|---|---
region_code|true|String|Country code|CN
status|true|String|SMS status.Please see details in appendix|accepted
price|true|float|the price billed by YCloud for the SMS sent|0.03
count|true|int|Quantity of SMS|1
mobile|true|String|Mobile number|+8615355498966
sid|true|String|Unique id with int64 value format|3310228982
create_timestamp|true|timestamp|The date and time using millisecond|1541580135423

**Response example:**
**Accpeted**

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

**Rejected**

```json

{
    "code": 2,
    "msg": "BAD_ARGUMENT The mobile is unverified"
}

```

#### 2.2 Delivery report

Your platform should respond with a 200 HTTP header. When our platform doesn't receive a 200 header, we will try to deliver the receipt again (up to 5 times),which are 1min 10min 60min 24h 72h.

Push method:

`POST  <callback_url>`

curl example：
`curl -H  -d'{"sid":"3310228982","status":"delivered","push_timestamp":1541580135423,"mobile":"+8615355498963"}' -X POST http://my.sms.com/call_back`

**Push parameters**

Paremeters|Required|Type|Description
---|---|---|---
sid|true|String|SMS id
status|true|String|SMS status
push_timestamp|true|timestamp|The date and time using millisecond
mobile|true|String|Mobile number
uid|false|String|
error_code|false|String
error_msg|false|String


#### 2.3 SMS records

`GET https://api.ycloud.com/v1/sms/list`

Request frequency limit:once every minute

`curl -H "Authorization:you_api_key" -X GET https://api.ycloud.com/v1/sms/list`


**Query**

Paremeters|Required|Type|Description
---|---|---|---
page|true|int|Page number starting from 1
page_size|true|int|Default value is 100,5000 at max
from|true|timestamp|Start time using millisecond
until|true|timestamp|End time using millisecond
sid|false|String|Other value will be ignored when sid is passed in.

**Response parameters**

Paremeters|Required|Type|Description
---|----|----|----
count|true|int|Quantity
page_no|true|int|Page number starting from 1
page_size|true|int|-
data|true|array\<Sms\>|The message object
msg|true|String|
code|true|int|Return status


**The message object**

Paremeters|Required|Type|Description|Example
---|---|---|---|---
region_code|true|String|Country code|CN
status|true|String|SMS status.Please see details in appendix|accepted
price|true|float|the price billed by YCloud for the SMS sent|0.03
unit|true|String|Currency|dollars
count|true|int|Quantity of sms|1
mobile|true|String|Mobile number|+8615355498966
sid|true|String|Unique id with int64 value format|3310228982
text|true|String|the content of SMS 
error_code|false|String|Error code when sms is undelivered|YC:NO BALANCE
create_timestamp|true|timestamp|The date and time of sms created,using millisecond|1541580135423
status_timestamp|false| timestamp|The date and time of delivery report,using millisecond| 1541580135423
push_timestamp |false| timestamp|The date and time of push,using millisecond| 1541580135423

**Response example 1**

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
**Response example 2 (no data):**

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


#### 2.4 Inbound SMS

Your platform should respond with a 200 HTTP header. When our platform doesn't receive a 200 header, we will try to push the message again (up to 5 times),which are 1min 10min 60min 24h 72h.


`POST <inbound_callback_url>`

`curl -d '{"id":"a1s1we1111","mobile":"+8615355498966","text":"Thank you","inbound_timestamp": 1541580135423}' -X POST http://my.sms.com/inbound_callback`


Paremeters|Required|Type|Description
---|---|---|---
id|true|String|Unique id
mobile|true|String|Mobile number
text|true|String|The content of SMS
inbound_timestamp|true| timestamp |The date and time



### 3.Account
#### 3.1 Balance API

`GET https://api.ycloud.com/v1/account/balance`

`CURL -H "Authorization:you_api_key" -X GET https://api.ycloud.com/v1/account/balance`

**Response parameters**

Paremeters|Required|Type|Description
---|---|---|---
count|true|int|Quantity of SMS
msg|true|String|
balance|true|Balance object|-


**The account object**

Paremeters|Required|Type|Description|Example
---|---|---|---|---
currency|true|String|Currency, USD and CNY only|USD
amount|true|float|Remaining balance and round off to 5 decimals|3.44


**Response example 1**

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

**Response example 2**

```json

{
	"amount":10000010190.07658,
	"currency":"USD"
}

```

### Appendix

#### Appendix 1.code

Code|Description|Detail
---|---|-----
-2|NO_BASE_PRICE
-1|INVALID_API_KEY
0|SUCCESS
1|ARGUMENT_MISSING
2|BAD_ARGUMENT
3|NO_BALANCE


#### Appendix 1.status 

Status|Description|
---|---
Unaccepted|Your request is rejected by YCloud
Accepted|YCloud has received your API request to send a message 
Failed|The message could not be sent. This can happen for reasons including insufficient balance or invalid channels. Error code: YC:XXXXXX
Delivering|YCloud is in the process of dispatching your message and waiting for delivery report to return
Delivered|YCloud has received confirmation of message delivery 
Undelivered|Failed to return by mobile carrier. Error code：OP:XXXXXXX（Including request to MC）
Expired|expire_secs expired,we were unable to deliver the message because the device was possibly not connected with A2P network.

#### Appendix 2.error_code

Prefix|Description
----|----
YC:|"YC prefix": YCloud
OP:|"OP prefix": Mobile carrier




**YCloud**

Error_code|Description
-----|-------
YC:13|营销短信暂停发送
YC:20|不支持的国家地区
YC:MOBILE BLACK	|号码在账号黑名单中，不扣费
YC:NO BALANCE|Insufficient balance and transaction failed
YC:NOOP|Invalid channel.YCloud will not charge the SMS. 
YC:NOSEND|无效通道，不扣费



**Mobile carrier**
OP:+Value returned by mobile carrier
 