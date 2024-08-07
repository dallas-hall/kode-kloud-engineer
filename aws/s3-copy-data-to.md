# AWS Copy Data To S3

## Task

> The Nautilus DevOps team is presently immersed in data migrations, transferring data from on-premise storage systems to AWS S3 buckets. They have recently received some data that they intend to copy to one of the S3 buckets.
>
> * S3 bucket named `nautilus-cp-6320` already exists. Copy the file `/tmp/nautilus.txt` to s3 bucket `nautilus-cp-6320`.

## Research

* AWS S3.
  * https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html
  * https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html or https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html

```bash
# Copy the file.
aws s3 cp /tmp/nautilus.txt s3://nautilus-cp-6320/nautilus.txt
```

```
upload: ../tmp/nautilus.txt to s3://nautilus-cp-6320/nautilus.txt
```
