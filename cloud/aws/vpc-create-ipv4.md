# AWS Create A VPC With IPv4

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> Create a VPC named `devops-vpc` in `us-east-1` region with any IPv4 CIDR block.

## Research

* AWS VPC
  * https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpcs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html

```bash
# Create VPC and IPv4 CIDR Block.
aws ec2 create-vpc --cidr-block 10.0.0.0/24 \
  --query Vpc.VpcId \
  --output text \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=devops-vpc}]'

# Describe VPC
aws ec2 describe-vpcs
```

```json
{
    "Vpcs": [
        {
            "CidrBlock": "10.0.0.0/24",
            "DhcpOptionsId": "dopt-0d075b71b8b246aa2",
            "State": "available",
            "VpcId": "vpc-0c997ee737aa8255c",
            "OwnerId": "471112877872",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0fe4f7f5e7d8ad1ed",
                    "CidrBlock": "10.0.0.0/24",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "devops-vpc"
                }
            ]
        },
        {
            "CidrBlock": "172.31.0.0/16",
            "DhcpOptionsId": "dopt-0d075b71b8b246aa2",
            "State": "available",
            "VpcId": "vpc-01aa1b915e70c4af9",
            "OwnerId": "471112877872",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0f9b42fb2083a30ce",
                    "CidrBlock": "172.31.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": true,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "KodeKloudVPC"
                }
            ]
        }
    ]
}
```
