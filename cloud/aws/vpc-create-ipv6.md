# AWS Create A VPC With IPv6

## Task

> The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the AWS cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. Their approach involves creating Virtual Private Clouds (VPCs) as the initial step, as they will be provisioning various services under different VPCs.
>
> Create a VPC named `xfusion-vpc` in `us-east-1` region with the Amazon-provided IPv6 CIDR block.

## Research

* AWS VPC
  * https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpcs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html

```bash
# Create VPC and IPv6 CIDR Block.
aws ec2 create-vpc --cidr-block 10.0.0.0/24 \
  --amazon-provided-ipv6-cidr-block \
  --query Vpc.VpcId \
  --output text \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=xfusion-vpc}]'
```

```
vpc-05c044cf0d33d82aa
```

```bash
# Describe VPC
aws ec2 describe-vpcs --vpc-ids vpc-05c044cf0d33d82aa
```

```json
{
    "Vpcs": [
        {
            "CidrBlock": "10.0.0.0/24",
            "DhcpOptionsId": "dopt-0b2a96f80821a9b46",
            "State": "available",
            "VpcId": "vpc-05c044cf0d33d82aa",
            "OwnerId": "905418195910",
            "InstanceTenancy": "default",
            "Ipv6CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0ba9f55997c402d97",
                    "Ipv6CidrBlock": "2600:1f18:75fe:3700::/56",
                    "Ipv6CidrBlockState": {
                        "State": "associated"
                    },
                    "NetworkBorderGroup": "us-east-1",
                    "Ipv6Pool": "Amazon"
                }
            ],
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-03e6143cd637444ef",
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
                    "Value": "xfusion-vpc"
                }
            ]
        }
    ]
}
```
