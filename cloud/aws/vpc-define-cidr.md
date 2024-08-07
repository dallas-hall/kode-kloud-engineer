# AWS Create A VPC With Custom IPv4 CIDR

## Task

> The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the AWS cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. Their approach involves creating Virtual Private Clouds (VPCs) as the initial step, as they will be provisioning various services under different VPCs.
>
> Create a VPC named `nautilus-vpc` in `us-east-1` region with `192.168.0.0/24` IPv4 CIDR.

## Research

* AWS VPC
  * https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpcs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/vpc/latest/userguide/create-vpc.html

```bash
# Create VPC and IPv4 CIDR Block.
aws ec2 create-vpc --cidr-block 192.168.0.0/24 \
  --query Vpc.VpcId \
  --output text \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=nautilus-vpc}]'

# Describe VPC
aws ec2 describe-vpcs
```

```json
{
    "Vpcs": [
        {
            "CidrBlock": "192.168.0.0/24",
            "DhcpOptionsId": "dopt-02e6fd26aa5d4c373",
            "State": "available",
            "VpcId": "vpc-0b9d0d57a99fa1954",
            "OwnerId": "905418106103",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0fdc8d00e1c87fa31",
                    "CidrBlock": "192.168.0.0/24",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "nautilus-vpc"
                }
            ]
        },
        {
            "CidrBlock": "172.31.0.0/16",
            "DhcpOptionsId": "dopt-02e6fd26aa5d4c373",
            "State": "available",
            "VpcId": "vpc-03dc471b7dc8b5066",
            "OwnerId": "905418106103",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0366afa3ab5cd7f63",
                    "CidrBlock": "172.31.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": true
        }
    ]
}
```
