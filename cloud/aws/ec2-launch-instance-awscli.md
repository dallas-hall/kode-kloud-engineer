# Launch EC2 Instance

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. Due to recent security restrictions have revoked AWS console access. Nevertheless, the team retains CLI access via the aws-client host (the landing host for this lab), allowing them to manage AWS resources effectively.
>
> Create an EC2 instance using AWS CLI under default VPC with the following details:
> 1. The name of the instance must be `nautilus-ec2`.
> 2. You can use the `ami-0cd59ecaf368e5ccf` AMI to launch this instance.
> 3. The Instance type must be `t2.micro`.
> 4. Create a new RSA key pair named `nautilus-kp`.
> 5. Attach the default (available by default) security group.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html

## Steps

Follow the steps at https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > nautilus-kp.pem

# Launch EC2.
aws ec2 run-instances --image-id ami-0cd59ecaf368e5ccf \
  --count 1 \
  --instance-type t2.micro \
  --key-name nautilus-kp \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]'
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0cd59ecaf368e5ccf",
            "InstanceId": "i-0774d1f7b047ae1f9",
            "InstanceType": "t2.micro",
            "KeyName": "nautilus-kp",
            "LaunchTime": "2024-08-08T04:52:06.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1d",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-32-199.ec2.internal",
            "PrivateIpAddress": "172.31.32.199",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-086f344bba1b6bceb",
            "VpcId": "vpc-023cd07feb2c6f3e2",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "461ea24c-4dca-440f-98df-3f17c7738f90",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2024-08-08T04:52:06.000Z",
                        "AttachmentId": "eni-attach-09ea13323d7489943",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-0a9d52ad4c69f5bda"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0e:50:c8:93:42:ab",
                    "NetworkInterfaceId": "eni-05f5b8ac88b782e8d",
                    "OwnerId": "211125465619",
                    "PrivateDnsName": "ip-172-31-32-199.ec2.internal",
                    "PrivateIpAddress": "172.31.32.199",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-32-199.ec2.internal",
                            "PrivateIpAddress": "172.31.32.199"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-086f344bba1b6bceb",
                    "VpcId": "vpc-023cd07feb2c6f3e2",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-0a9d52ad4c69f5bda"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "nautilus-ec2"
                }
            ],
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        }
    ],
    "OwnerId": "211125465619",
    "ReservationId": "r-0bd534bace4b58abc"
}
```

</details>

```bash
# Check EC2 status.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1d",
            "InstanceId": "i-0774d1f7b047ae1f9",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "InstanceStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "initializing"
                    }
                ],
                "Status": "initializing"
            },
            "SystemStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "initializing"
                    }
                ],
                "Status": "initializing"
            }
        }
    ]
}
```

Wait a few minutes for it to start up.

```bash
# Check EC2 status.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1d",
            "InstanceId": "i-0774d1f7b047ae1f9",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "InstanceStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            },
            "SystemStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            }
        }
    ]
}
```