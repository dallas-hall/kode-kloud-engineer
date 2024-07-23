# AWS Create Security Group

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> For this task, create a security group under default VPC with the following requirements:
> * Name of the security group is `devops-sg`.
> * The description must be `Security group for Nautilus App Servers`
> * Add the inbound rule of type `HTTP` with port range of `80` Enter the source CIDR range of `0.0.0.0/0`.
> * Add another inbound rule of type `SSH`, with port range of `22`. Enter the source CIDR range of `0.0.0.0/0`.

## Research

* Security Groups:
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#creating-security-group
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#creating-security-group or https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html and https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html

```bash
# Create the Security Group.
aws ec2 create-security-group \
--group-name devops-sg \
--description "Security group for Nautilus App Servers"
```

```json
{
    "GroupId": "sg-0fa20684c4745174b"
}
```

```bash
# Create the Security Group HTTP ingress rules.
aws ec2 authorize-security-group-ingress \
    --group-id sg-0fa20684c4745174b \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00aaa3d71f2a0c52a",
            "GroupId": "sg-0fa20684c4745174b",
            "GroupOwnerId": "211125650496",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

```bash
# Create the Security Group SSH ingress rules.
aws ec2 authorize-security-group-ingress \
    --group-id sg-0fa20684c4745174b \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-05c6fa3ae814ade7e",
            "GroupId": "sg-0fa20684c4745174b",
            "GroupOwnerId": "211125650496",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```
