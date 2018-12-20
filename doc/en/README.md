# YCloud api-doc

### 1. Overview

Your need to include following in your request headers:`Content-Type:application/json;charset=utf-8;`

#### 1.1 Authentication

With each API call, you will need to include your access key to authenticate yourself. `Authorization:you_api_key`

#### 1.2 Request structure

Your platform should respond with a 200 HTTP header. 

Basic reponse value:

Name|Type|Description
---|---|----
code|int|When code=0, successfully requsted.When code!=0,exceptions that something goes wrong
msg|string|When successfully requsted,it will show "success". Failed description when request failed.

Other returned value:
When the parameter is not found,it will not be included in the JSON returned values, meaning the filed is ignored when JSON serialization.
