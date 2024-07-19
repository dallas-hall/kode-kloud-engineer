# AWS Attach Elastic IP To EC2 Instance

## Task

> The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
>
> * There is an instance named `xfusion-ec2` and an `elastic-ip` named `xfusion-ec2-eip` in `us-east-1 region`. Attach the `xfusion-ec2-eip` elastic-ip to the `xfusion-ec2` instance.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* AWS attach Elastic IP to EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating
