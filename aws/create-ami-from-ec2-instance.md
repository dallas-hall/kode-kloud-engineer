# AWS Create Key Pair

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> For this task, create an AMI from an existing EC2 instance named `datacenter-ec2` with the following requirement:
> * Name of the AMI should be `datacenter-ec2-ami`, make sure AMI is in available state.

## Research

* AWS create key pair.
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > nautilus-kp.pem
```
