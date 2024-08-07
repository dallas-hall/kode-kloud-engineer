# AWS Delete A VPC

## Task

> The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the AWS cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. They created some services in different regions and later found that some of those can be deleted now.
>
> Delete a VPC named `xfusion-vpc` present in `us-east-1` region

## Research

* AWS VPC
  * https://docs.aws.amazon.com/vpc/latest/userguide/delete-vpc.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-vpc.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpcs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/vpc/latest/userguide/delete-vpc.html

```bash
# Describe VPC
aws ec2 describe-vpcs
```

```json
{
    "Vpcs": [
        {
            "CidrBlock": "172.31.0.0/16",
            "DhcpOptionsId": "dopt-0429ae4576af02ca8",
            "State": "available",
            "VpcId": "vpc-019bd0e468f8458bb",
            "OwnerId": "533266994003",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0fc04c50d9b66d006",
                    "CidrBlock": "172.31.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": true
        },
        {
            "CidrBlock": "10.0.0.0/16",
            "DhcpOptionsId": "dopt-0429ae4576af02ca8",
            "State": "available",
            "VpcId": "vpc-0f7f282da57f9a956",
            "OwnerId": "533266994003",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-084f6ebd39f2bf0cc",
                    "CidrBlock": "10.0.0.0/16",
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

```bash
# Delete VPC.
aws ec2 delete-vpc --vpc-id vpc-0f7f282da57f9a956
```
