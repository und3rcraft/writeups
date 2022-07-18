# Challenge 1

Our Security Team found a public bucket named **s3-challenge-1** during their initial enumeration phase. They now need your help to get flag.

Can you help them get the job done?

You can submit the flag to us on twitter at [@securitylabs_](http://twitter.com/securitylabs_). We have some hidden surprises for you!

For more such interesting labs and content register for beta at [https://www.securitylabs.tech](https://www.securitylabs.tech/)

Have fun hacking cloud 🙂


# Solution:
You can gobuster in s3 mode to find the s3 bucket with s3-challenge-1 being in the wordlist or just guess it.

gobuster s3 -w cloudlist.txt

https://s3-challenge-1.s3.amazonaws.com/

Viewing the link the bucket lists a file flag.txt

Viewing the file shows you:
 This is not the flag you want, but try harder. Maybe look at utUeH4svvjp1dJJiBCgS6Jpto4ilJASt :)

Having a look around it seems to follow the format of s3 bucket versioning.
https://docs.aws.amazon.com/AmazonS3/latest/userguide/versioning-workflows.html
https://docs.aws.amazon.com/AmazonS3/latest/userguide/RetrievingObjectVersions.html

Allows you to access the prior file version at:
https://s3-challenge-1.s3.amazonaws.com/flag.txt?versionId=utUeH4svvjp1dJJiBCgS6Jpto4ilJASt

Alternatively if the initial file didn't give you a hint to the version id. As s3 list, read and ListBucketVersions are being made available to the public, you can view the versions available for an s3 bucket via cli/s3api using any valid AWS cli account.

```
aws s3api list-object-versions --bucket s3-challenge-1
{
    "Versions": [
        {
            "ETag": "\"60aa6c807e6963ed19281b4b692b7b55\"",
            "Size": 97,
            "StorageClass": "STANDARD",
            "Key": "flag.txt",
            "VersionId": "lQD6sHemiK9A2bDILNXM.ls45QxTGqqw",
            "IsLatest": true,
            "LastModified": "2022-02-07T20:16:15+00:00"
        },
        {
            "ETag": "\"30df7577e8b91ed686cdef4587d2c014\"",
            "Size": 216,
            "StorageClass": "STANDARD",
            "Key": "flag.txt",
            "VersionId": "utUeH4svvjp1dJJiBCgS6Jpto4ilJASt",
            "IsLatest": false,
            "LastModified": "2022-02-07T20:14:11+00:00"
        }
    ]
}
```
