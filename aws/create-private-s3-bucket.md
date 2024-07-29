# AWS Create Private S3 Bucket

## Task

> As part of the data migration process, the Nautilus DevOps team is actively creating several S3 buckets on AWS. They plan to utilize both private and public S3 buckets to store the relevant data. Given the ongoing migration of other infrastructure to AWS, it is logical to consolidate data storage within the AWS environment as well.
>
> Create S3 bucket with the following details:
> 1. The name of the S3 bucket must be `devops-s3-16192`.
> 2. The S3 bucket must block all public access, i.e it must be a private bucket.

## Research

* AWS create s3 bucket
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html
  * https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/create-bucket.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html or https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/create-bucket.html

```bash
# Create the s3 bucket.
aws s3api create-bucket \
    --bucket devops-s3-16192 \
    --region us-east-1 \
    --acl private
```

```json
{
    "Location": "/devops-s3-16192"
}
```
