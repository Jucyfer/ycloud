# ycloud api-doc

### 1. 接口概述

所有请求http请求头部设置`Content-Type:application/json;charset=utf-8;`

#### 1.1 接口请求校验

apikey在http头部放置`Authorization:you_api_key`

#### 1.2 接口响应结构说明

http 返回200说明请求正常到达服务器.

接口基础返回值:

名称|类型|描述
---|---|----
code|int|code=0时,请求成功,code!=0时,说明请求异常
msg|string|请求成功时"success",失败时,为失败的描述

其他返回值:
当业务参数不存在时,返回json中没有此字段,即当字段为null时,json序列化时忽略此字段.
