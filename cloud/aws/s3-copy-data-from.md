# AWS Copy Data From S3

## Task

> The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their AWS account. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their AWS environment.
>
> A s3 bucket named `xfusion-bck-3456` already exists.
> 1. Copy the contents of `xfusion-bck-3456` s3 bucket to `/opt` directory on `aws-client` host (the landing host once you load this lab).
> 2. Delete the s3 bucket `xfusion-bck-3456`.

## Research

* AWS S3.
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html
  * https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/delete-bucket.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html or https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html and https://docs.aws.amazon.com/AmazonS3/latest/userguide/delete-bucket.html

```bash
# Copy everything from s3.
aws s3 cp s3://xfusion-bck-3456/ /opt --recursive
```

```
download: s3://xfusion-bck-3456/xfusion.txt to ../opt/xfusion.txt
```

```bash
# Delete s3
aws s3 rb s3://xfusion-bck-3456 --force
```

```
delete: s3://xfusion-bck-3456/xfusion.txt
remove_bucket: xfusion-bck-3456
```
