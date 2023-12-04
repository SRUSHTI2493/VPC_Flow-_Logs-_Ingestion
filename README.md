# 🚀 AWS VPC Flow Logs Ingestion 

VPC Flow Logs is a feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC. Flow logs can help you with a number of tasks, such as:

Monitoring the traffic that is reaching your instance Determining the direction of the traffic to and from the network interfaces Analyzing properties such as IP addresses, ports, protocol and total packets sent without the overhead of taking packet captures Flow log data is collected outside of the path of your network traffic, and therefore does not affect network throughput or latency. You can create or delete flow logs without any risk of impact to network performance.

This quickstart is a guide for ingestion AWS VPC Flowlogs into Snowflake. It demonstrates configuration of VPC flowlogs on AWS, ingestion using an external stage with Snowpipe and sample queries for CSPM and threat detection.

# ➡️ Prerequisites

▪️ AWS user with permission to create and manage IAM policies and roles

▪️ Snowflake user with permission to create tables, stages and storage integrations as well as setup snowpipe.

▪️ An S3 Bucket

# 🧩 Architecture

![image](https://github.com/SRUSHTI2493/VPC_Flow-_Logs-_Ingestion/assets/87080882/f508f885-d966-4f3e-9540-2011bf071403)

### Enable VPC Flow Logs and Push to S3

 From the VPC page in the AWS console, select the VPC you wish to enable flow logs on. Select the "Flow Logs" tab and press "Create flow log"
 ![image](https://github.com/SRUSHTI2493/VPC_Flow-_Logs-_Ingestion/assets/87080882/80cb647f-d7e5-4b82-9ae9-8e7305bc8bfe)

Configure VPC flow logs as desired. Use the following settings:

 **Destination:**   Send to an Amazon S3 Bucket

 **S3 Bucket ARN:** S3 Bucket ARN and prefix of existing bucket ( or press the "create s3 bucket" link to create a new one)

 **Log file format:** Parquet

 ![image](https://github.com/SRUSHTI2493/VPC_Flow-_Logs-_Ingestion/assets/87080882/21c0e891-2d23-4946-b75a-55d0899f2b94)

**1.Create one IAM Role and Paste ARN in the below code.**

**2.Create one S3 Bucket and Paste It's URI in the code below**

### Create a storage integration in Snowflake
```
create STORAGE INTEGRATION s3_int_vpc_flow
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::<AWS_ACCOUNT_NUMBER>:role/<RoleName>'
  STORAGE_ALLOWED_LOCATIONS = ('s3://<BUCKET_NAME>/<PREFIX>/');

DESC INTEGRATION s3_int_vpc_flow;
```
Take note of **STORAGE_AWS_IAM_USER_ARN** and **STORAGE_AWS_EXTERNAL_ID**

![image](https://github.com/SRUSHTI2493/VPC_Flow-_Logs-_Ingestion/assets/87080882/9f7cde7f-8869-4455-b1b8-37e20efd958e)

Open up Cloudshell in the AWS console by pressing the aws cloudshell icon icon on the right side of the top navigation bar or run the following commands in your terminal once configured to use the AWS CLI.

Export the following variables, replacing the values with your own.

```
export BUCKET_NAME='<BUCKET_NAME>'
export PREFIX='<PREFIX>' # no leading or trailing slashes
export ROLE_NAME='<ROLE_NAME>'
export STORAGE_AWS_IAM_USER_ARN='<STORAGE_AWS_IAM_USER_ARN>'
export STORAGE_AWS_EXTERNAL_ID='<STORAGE_AWS_EXTERNAL_ID>'

```
**Open your IAM role and edit Trust (Trust relationships) Policy and paste below code in it.**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "AWS": "'${STORAGE_AWS_IAM_USER_ARN}'"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "'${STORAGE_AWS_EXTERNAL_ID}'"
                }
            }
        }
    ]
}

```
**Now We will require to create an Inline Policy for our role**
