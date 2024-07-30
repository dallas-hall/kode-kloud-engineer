# AWS Apply S3 Versioning

## Task

> Data protection and recovery are fundamental aspects of data management. It's essential to have systems in place to ensure that data can be recovered in case of accidental deletion or corruption. The DevOps team has received a requirement for implementing such measures for one of the S3 buckets they are managing.
>
> * The s3 bucket name is `datacenter-s3-5764`, enable `versioning` for this bucket.

## Research

* AWS create key pair.
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/manage-versioning-examples.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonS3/latest/userguide/manage-versioning-examples.html

```bash
# Enable versioning.
aws s3api put-bucket-versioning --bucket datacenter-s3-5764 --versioning-configuration Status=Enabled
```
