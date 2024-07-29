# AWS Create Public S3 Bucket

## Task

> As part of the data migration process, the Nautilus DevOps team is actively creating several S3 buckets on AWS. They plan to utilize both private and public S3 buckets to store the relevant data. Given the ongoing migration of other infrastructure to AWS, it is logical to consolidate data storage within the AWS environment as well.

Create a public s3 bucket named `devops-s3-15326`.

## Research

* AWS create s3 bucket
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html
  * https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/create-bucket.html
  * https://docs.aws.amazon.com/cli/latest/reference/s3api/delete-public-access-block.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html or https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/create-bucket.html and https://docs.aws.amazon.com/cli/latest/reference/s3api/delete-public-access-block.html

```bash
# Create the public s3 bucket.
aws s3api create-bucket \
    --bucket devops-s3-15326 \
    --region us-east-1
```

```json
{
    "Location": "/devops-s3-15326"
}
```

```bash
# Delete public access block
aws s3api delete-public-access-block \
    --bucket devops-s3-15326
```
