# Challenge 6
---
Challenge-6

We are back with another challenge of the week. Our Security team discovered a public API gateway with alleged connections with a lambda.

“Team found an API Gateway with Id ‘**543vvbi1bd**‘ publicly accessible in **us-east-2** region. There are allegations that you cant really bruteforce to find the right Endpoint and hence we have these access keys for you“

AWS_ACCESS_KEY_ID : AKIA\<redacted>OBQ  
AWS_SECRET_ACCESS_KEY : utIpl\<redacted>4rf6

You can DM us the flag at [@securitylabs_](http://twitter.com/securitylabs_) to verify it. We might have some surprise for you!

_For more such interesting labs and content register for beta at [https://securitylabs.tech](https://securitylabs.tech)_

Have fun hacking cloud with SecurityLabs




---


## Solution
Based on the syntax of aws api gateways we know the URL should be https://543vvbi1bd.execute-api.us-east-2.amazonaws.com

The challenge suggest there might be a path or directory that is hidden.

Using AWS CLI we query the routes

```
aws apigatewayv2 get-routes --api-id 543vvbi1bd --profile cloudchall
{
    "Items": [
        {
            "ApiKeyRequired": false,
            "AuthorizationScopes": [],
            "AuthorizationType": "NONE",
            "RequestParameters": {},
            "RouteId": "eq86rvc",
            "RouteKey": "GET /a4dc93da2274047f6a35ed800144419e",
            "Target": "integrations/ra7ynoq"
        }
    ]
}
```

Visiting https://543vvbi1bd.execute-api.us-east-2.amazonaws.com/a4dc93da2274047f6a35ed800144419e gives us the flag.
```

HTTP/2 200 OK
Date: Tue, 15 Mar 2022 23:47:32 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 218
Apigw-Requestid: PDLPHjNxiYcEMXg=

flag{a<redacted>s}
 You can find more interesting challenges and learning materials at SecurityLabs. Register for beta at https://www.securitylabs.tech in case you haven't.
 Have fun hacking cloud with SecurityLabs
 ```
