# AWS Subnet

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition.
>
> For this task, create one subnet named `xfusion-subnet` under default VPC.

## Research

* AWS create Subnet.
  * https://docs.aws.amazon.com/vpc/latest/userguide/create-subnets.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-subnet.html

## Steps

Follow the steps at https://docs.aws.amazon.com/vpc/latest/userguide/create-subnets.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/create-subnet.html

```bash
# Get VPC ID
aws ec2 describe-vpcs
```

```json
{
    "Vpcs": [
        {
            "CidrBlock": "172.31.0.0/16",
            "DhcpOptionsId": "dopt-0e9e91e4b4e5a33be",
            "State": "available",
            "VpcId": "vpc-00cfd39195dea5221",
            "OwnerId": "730335390054",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0f773c4c7368401a3",
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

```bash
# Get existing CIDRs
aws ec2 describe-subnets | grep -i \"cidrblock
```

```
            "CidrBlock": "172.31.80.0/20",
            "CidrBlock": "172.31.48.0/20",
            "CidrBlock": "172.31.0.0/20",
            "CidrBlock": "172.31.16.0/20",
            "CidrBlock": "172.31.32.0/20",
            "CidrBlock": "172.31.64.0/20",
```

```bash
# View next available host.
ipcalc 172.31.80.0/20
```

```
Network:	172.31.80.0/20
Netmask:	255.255.240.0 = 20
Broadcast:	172.31.95.255

Address space:	Private Use
HostMin:	172.31.80.1
HostMax:	172.31.95.254
Hosts/Net:	4094

```

The last host is 172.31.95.254 and the broadcast is 172.31.95.255 so we can use 172.31.96.0 for our subnet.

```bash
# Create the Subnet
aws ec2 create-subnet \
    --vpc-id vpc-00cfd39195dea5221 \
    --cidr-block 172.31.96.0/20 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-subnet}]'
```

```json
{
    "Subnet": {
        "AvailabilityZone": "us-east-1e",
        "AvailabilityZoneId": "use1-az3",
        "AvailableIpAddressCount": 4091,
        "CidrBlock": "172.31.96.0/20",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-06d77ee01f9bcedb2",
        "VpcId": "vpc-00cfd39195dea5221",
        "OwnerId": "730335390054",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "xfusion-subnet"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:730335390054:subnet/subnet-06d77ee01f9bcedb2",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}
```
