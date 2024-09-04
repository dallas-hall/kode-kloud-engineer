# AWS Create Elastic IP

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> For this task, allocate an Elastic IP address, name it as `nautilus-eip`.

## Research

* AWS Elastic IP
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-allocating
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-address.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-allocating or https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-address.html

```bash
# Create the key pair
aws ec2 allocate-address \
    --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=nautilus-eip}]'
```

```json
{
    "PublicIp": "3.209.43.255",
    "AllocationId": "eipalloc-0f8d706294a8d2dfa",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-1",
    "Domain": "vpc"
}
```