# ssm-ami-encryption-automation
Automated AMI encryption of updated Amazon provided AMIs using SSM

The idea is to auto create a new encrypted AMI every time Amazon updates an AMI such as Amazon Linux 2 or Windows 2016 Base.

Current included Linux/Windows Versions, US-East-1
---
- Amazon Linux 2
- Windows 2012 R2 Base
- Windows 2016 Full Base
- Windows 2016 SQL 2017 Standard


Workflow of the automation
---

1. The lambda function is subscribed to an official AWS provided SNS topic:
`arn:aws:sns:us-east-1:137112412989:amazon-linux-ami-updates` # Amazon linux
`arn:aws:sns:us-east-1:801119661308:ec2-windows-ami-update`   # Windows
2. When one of the SNS topics are published too, lambda is triggered to look for the latest related AMI and sends to SSM document
3. The SSM automation document(s) are kicked off with the latest AMI and begins the encryption process
    1. The Linux encryption uses `createEncryptedCopy` directly.
    2. Windows needs to spin up an instance first.
        1. We run `sysprep` to make sure the custom AMI can get Admin PWs and run user data scripts.
        2. Then create an image from that before finally creating a new encrypted a new image.
        3. The temporary Windows instance is terminated as part of the SSM automation.
        4. We also clean up the pre-encrypted image that was created.
4. Once completed the newly encrypted AMI(s) are available.


Creating stack
---

1. Create an S3 bucket to store the templates, example `foobucket`
2. Upload all CloudFormation templates except `main.yml` and `sns-subscription.yml` to the s3 bucket
3. Run `main.yml` in US-East-1 region and provide s3 bucket name, `foobucket`, where the CloudFormation templates were uploaded
4. Note the `xSSMLambdaARN`s provided as output from the `main.yml` run
5. Run `sns-subscription.yml` in the US-East-1 region and specify lambda ARNs from `xSSMLambdaARN` output created in step 3


*Forked from https://github.com/b-b3rn4rd/ssm-ami-automation
