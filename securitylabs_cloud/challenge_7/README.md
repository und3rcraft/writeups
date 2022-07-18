# Challenge 7
---
We are back with another challenge of the week. Our Security team discovered some access keys belonging to a lambda function called `myFunction` in `us-east-2` region. Help them and get the flag.

```
AKIA<redacted>X
XFzn<redacted>/7PS
```

You can DM us the flag at [@securitylabs_](http://twitter.com/securitylabs_) to verify it. We might have some surprise for you!

_For more such interesting labs and content register for beta at [https://securitylabs.tech](https://securitylabs.tech/)_

Have fun hacking cloud with SecurityLabs 🙂



---


## Solution
We are given a key, and a function name to start so lets check out what those tell us.

```
aws sts get-caller-identity

{

 "UserId": "AIDAYQDT3AH74XLQPO7LM",

 "Account": "584358494719",

 "Arn": "arn:aws:iam::584358494719:user/challenge-7"

}
```

Checking if we can get the function and code
```
aws lambda get-function --function-name myFunction --region us-east-2

{

 "Configuration": {

 "FunctionName": "myFunction",

 "FunctionArn": "arn:aws:lambda:us-east-2:584358494719:function:myFunction",

 "Runtime": "python3.8",

 "Role": "arn:aws:iam::584358494719:role/service-role/myFunction-role-pg5cgb2a",

 "Handler": "lambda_function.lambda_handler",

 "CodeSize": 255,

 "Description": "",

 "Timeout": 3,

 "MemorySize": 128,

 "LastModified": "2022-03-22T15:07:47.000+0000",

 "CodeSha256": "cVannvj2y0Gc/qmDl1t6W3IvDil30MgpftnuZXVgR/0=",

 "Version": "$LATEST",

 "TracingConfig": {

 "Mode": "PassThrough"

 },

 "RevisionId": "875b1042-fa6f-49e2-bfd8-36c061b0df61",

 "Layers": [

 {

 "Arn": "arn:aws:lambda:us-east-2:584358494719:layer:challenge-7:1",

 "CodeSize": 378

 }

 ],

 "State": "Active",

 "LastUpdateStatus": "Successful",

 "PackageType": "Zip",

 "Architectures": [

 "x86_64"

 ]

 },

 "Code": {

 "RepositoryType": "S3",

 "Location": "https://awslambda-us-east-2-tasks.s3.us-east-2.amazonaws.com/snapshots/584358494719/myFunction-2b2eb97e-2758-49b2-9981-932d4f2bd567?versionId=SBmsIao3tPl41Kk_IkcHjzkvJYb7QDKW&X-Amz-Security-Token=IQoJb3JpZ2luX2VjELD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMiJHMEUCIQD2U0qI55n6XGP1QiuDwKemoJNuaDeBt9sHI9tSrvOC0QIgbcCQnP0OzbFbhTRsPFZGR9K21YH7JguN7iT6KlzwuZoq%2BgMIORADGgwxMDQyNDYwMTc4NjUiDJxHrs27sLCFO68hIyrXA1Z162rWKeoX7%2FYMlUjGjQ0D%2F3yocsRdvlG2qeFHmGBrKTguL1Nd4tq6bGzM7aQEkrycB%2FtVsv6pwLkA7YRLAkMa1no5cb5CkuvV%2FpO%2Bb%2BIUw01w2AgSdmtTj0sxn8OWg%2FSRJUKit3r%2FvtDPfbp08XlkUsXgObqcSHj0U6EXUiyqnmjNA96Qe7IjxWXU6pM1RvRecxlJHbvQToVE%2BgIko073em0maDGQRg9uP3P9AGlIngB1HHKHArbPQqdoZFsV5j%2FftevUQTIAhdVUrZVIip13pfs2g9ZLs2xOJlLBjLiZTGFnV7WGmopgvT%2BqATXQ8Bu5nVxOW76Pj%2FHVnVLSj%2FekSo2zVRQ2D9fyZZpkNN6z%2BDjhfOmmXLahNN2y9w6NvOi3GCE6%2BL0piUxLBNtm8aFW3i4CnADdma0cGeXQojwsNndUyHXLKbayGxrL5HaS9mB6FlF6C0oaOwT4HARkGcQDPJJdWtdc4bk0yn5Rdp761hTTQMkXANI3RIt%2FxwLlGUY3f%2BxEguxONBf3abxx00ysrEKVcLOAvow3jmG8%2ByH9Gahc5tjHZ3yWsOY1E8qUdOgA8wlexbRuFupZmebr%2BsJVvTqKterrWE231TR7jwfdHNCfYM%2B%2FHjDLzemRBjqlAdl6lugu%2F3Xim%2Bm7Ofo3fnMwvmrF6eBL8XbLWhgLLsJBfUOo2r0opwLeg7zS5V68ytV2IW%2FdRiRAMYLwbdA5sSQF2WcP1XXhb09x1venANgh4lh8U6Gc5xO%2FMywKjtKNCMmgZQ9hSghAC3Sq%2FH0hpRUMrfwb3SHHA2GBtsf4CttmuJ1gk6wpdc%2FLkAAw0sFjgZ2eYL1%2F6ucbO8Upx1IxoLoKWwruYA%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220323T005357Z&X-Amz-SignedHeaders=host&X-Amz-Expires=599&X-Amz-Credential=ASIARQRML75E5MM2A7UW%2F20220323%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Signature=3a38603eb915fac13b2f0bf85596e073be7d43cf9631e8298756e49b82ef61b9"
```

Downloading the code
```
from seclabs import get_flag

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

We can see a library/dependencies is being used, these can be supplied to the Lambda via inclusion in the initial zip file or via layers.
Layers allow sharing the libraries amongst multiple Lambda functions. Since we didn't get the library via download the source code, it is likely in a layer.


With the initial get function we see
```
"Layers": [

 {

 "Arn": "arn:aws:lambda:us-east-2:584358494719:layer:challenge-7:1",

 "CodeSize": 378

 }
```

We can grab this and download the code in a similar way to the initial function
```
aws lambda get-layer-version-by-arn --arn arn:aws:lambda:us-east-2:584358494719:layer:challenge-7:1 --region us-east-2

{

 "Content": {

 "Location": "https://awslambda-us-east-2-layers.s3.us-east-2.amazonaws.com/snapshots/584358494719/challenge-7-1652086d-a340-4c0f-be18-125805a77101?versionId=SY3K0bK_kUT0MB9a.g7iHwoUsEVU3ZG7&X-Amz-Security-Token=IQoJb3JpZ2luX2VjELD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMiJHMEUCIQClKUP3%2FkU3zk0df%2FzvpHkWpi9TZj6BWWtRHq%2FQPbMYBAIgf5PItgVPsbJuSPbUiAd8ilmI5bzL09%2Fdg6KHbwNg9lQq%2BgMIORADGgwxMDQyNDYwMTc4NjUiDADk2flhy1crCfiRRSrXA4ihgKJbaiAOF4ISV%2F0JEPSbZz0tGamHTAJmfz0BuWggCXYDinELeh5Im%2BSnC7%2FpyTOCOOWHDykdUungweK%2Fgt%2B96TdFdLXEGp1Q3kqvKdK8LlsmXoDLfi8%2F9DvPHzW3dF8kRQTGzeSAP0eHLxjS3XdK9S%2FG%2BA2Rs%2F7HeDmEfbanhk%2B1rsYpmOaC7nCYF8BA9oAV1nrNwCceCekUcxjGaVrKfXMb0bb0hey4D%2BvyfiWwlDd%2F06bX%2FVZDzOE333D6l1SEKOVtwpLiqG%2FROeNMyMNNCTu%2Bxvp2AUp2xAkK8SHnBMQ9Yv7YwlksNb6Z1TVD9IdXwM8CHvTs73wD26Zkj9dvXeoHwHGxUTTvnnYcFkKUznQOXqsfXWVuKI7h%2BJsAO%2BbDImw1C4gddAVc5LKei0npSOsIUAy8r%2FaCT251EOitonlk%2BqsaBvrNnXhuKe2Hw7oeO2u5CEQfZz%2FNIM%2BtM4nDT4QWFKhdyIJejzWGnJD%2F%2FGdAHTkNYx%2FJYVN4fEgYWtpoLZ7Z2JrW2hYSICE9fXbkPzl4a8REyVujJLbT3X3XmwQ90aW7gHLOc9DlpfBcqlJs48eLPKKUDCUbsFzh2A7DmUpe6S5a9LoqPzKwalP9ziCHIfsnMzC8xemRBjqlAZHRPPdhkE%2FejOwfrTr2lJTGU59YFHXRCMWv4mKiL1%2Fhtng9kQO02bcCYz9FuO3hJydD8toIQuAS0T5dyXowZ8ejYffMt509A4bOCSoJU4LvAE28gTpjGlOASkKr6QUKNgFN7hWfI04NSBgzbQ%2FDBBW5PYUpC2LSRj42pHJwnUlhwUK72hicNjViixOZiyAVVepsi9L%2FCadH1FH9RKWL3%2FS8qRQhsQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220323T010932Z&X-Amz-SignedHeaders=host&X-Amz-Expires=600&X-Amz-Credential=ASIARQRML75EVFCOIHZ4%2F20220323%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Signature=72776221037cfe963641b271c9e965bf18d1dde171075bd19701356b71a9d9c6",

 "CodeSha256": "uDGol4FCXfW1uB2Yj9mxTiIbKMNZOXDLR/gleWJOlc4=",

 "CodeSize": 378

 },

 "LayerArn": "arn:aws:lambda:us-east-2:584358494719:layer:challenge-7",

 "LayerVersionArn": "arn:aws:lambda:us-east-2:584358494719:layer:challenge-7:1",

 "Description": "",

 "CreatedDate": "2022-03-22T15:06:44.178+0000",

 "Version": 1,

 "CompatibleRuntimes": [

 "python3.7",

 "python3.9",

 "python3.8"

 ],

 "CompatibleArchitectures": [

 "x86_64"

 ]

}
```

```
cat seclabs.py

def get_flag():

 flag = "flag{<redacted>}\n You can find more interesting challenges and learning materials at SecurityLabs. Register for beta at https://www.securitylabs.tech in case you haven't\n

Have fun hacking cloud with SecurityLabs"



 return flag



def get_hash_flag(flag)

 return flag
```
