# Lambda_DaylightSavingsUpdate

This Lambda is using the [Bash2 \[1\]](https://github.com/dubrowin/Bash2-Lambda/) Layer which is an update to the [Bash Layer by gkrizek \[2\]](https://github.com/gkrizek/bash-lambda-layer) for Amazon Linux 2.

# Prerequisites

## Timezone File

Before you start, you'll need a timezone file. The binary zdump is not available in the Lambda runtime, but it is available on the Cloudshell interface on the console. To create the Timezone file, you'll need an S3 bucket. To create the file execute the following command for your Timezone. Timezones can be found in /usr/share/ on Linux based systems like Cloudshell.

For Jerusalem:
 ```zdump /usr/share/zoneinfo/Asia/Jerusalem > /tmp/timezones-data.txt```

For New York:
 ```zdump /usr/share/zoneinfo/America/New_York > /tmp/timezones-data.txt```
 
 This timezone file has the Daylight savings time changes up to 2499. If you need beyond this date, you'll need to run this command again with updated timezone information.

Next copy this timezone file to your bucket. The data file should be around 240k, so the cost is near $0.

 ```aws s3 cp /tmp/timezones-data.txt s3://<bucket>/```

## Permissions
 
### S3 Bucket
 Your Lambda will need read-only permission to your bucket and the object from above. 
  
### CloudWatch Events

  Your Lambda will need permission to change the execution times on the desired events. I found this policy to work affectively, ***remember*** to include the CloudWatch Event Rule that will execute your Lambda that does the changes.
  
  ```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "events:DescribeRule",
                "events:PutRule"
            ],
            "Resource": [
                "arn:aws:events:<region>:<account id>:rule/DaylightSavingsUpdate",
                "arn:aws:events:<region>:<account id>:rule/RuleToUpdate1",
                "arn:aws:events:<region>:<account id>:rule/RuleToUpdate2"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "events:ListRules",
            "Resource": "*"
        }
    ]
 }
```
## Putting it all Together

So now you are ready to create your Bash2 Lambda.
- Upload the Bash2 Layer to the region you will run the Lambda
- Create a Custom Runtime Lambda for Amazon Linux 2, bootstrap included
- Add the IAM policies that allow for S3 Bucket Access and CloudWatch Event Rule modifications.
- Configure the RAM to 512 (just to be safe) and the timeout to 1 min
- Change ```bootstrap.sample``` to ```boostrap```
- Change ```hello.sh.sample``` to ```hello.sh```
- Paste the code from this GitHub Repo
 - Modify the ```LIBBUCKET``` variable to be your S3 Bucket
 - Modify the ```LIB``` variable if you changed the name of your ```timezones-data.txt``` data file
 - Modify ```EVENTS``` to include the CloudWatch Event Rules to be updated
 - Modify ```ME``` to be the CloudWatch Event Rule that executes your Lambda

You should be ready to test

**NOTE:** The script does not currently handle switching days. So if you need the add and/or remove hour to change days, the API call will probably fail.

[1] https://github.com/dubrowin/Bash2-Lambda/
[2] https://github.com/gkrizek/bash-lambda-layer
