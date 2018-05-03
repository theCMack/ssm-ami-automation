# ssm-encrypt-ami-automation
Automated AMI encryption creation using SSM

Forked from https://github.com/b-b3rn4rd/ssm-ami-automation to create automation around encrypting new AMIs as they are published by AWS. The idea is that when AWS publishes an updated AMI via SNS then we'll create and encrypted copy and publish it.

Workflow
---------

1. The  lambda function is subscribed to the official SNS topic
`arn:aws:sns:us-east-1:137112412989:amazon-linux-ami-updates`
2. The SSM automation document is triggered with the latest AMI  when a new SNS notification is received
3. Automation document launches encrypts new AMI
4. CloudWatch Event gets notified when the SSM has finished or failed the execution. It sends a message to the SNS topic which is subscribed by the notification email.

Creating stack
-------------

1. Upload all cloudformation templates except `main.yml` and `sns-subscription.yml` to the s3 bucket
2. Run `main.yml` in the desired region and provide s3 bucket name where cloudformations were uploaded
3. Run `sns-subscription` in the us-east-1 region and specify lambda ARN  from `SSMLambdaARN` output created in step 2
