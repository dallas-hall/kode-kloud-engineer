# Modify EC2 Instance Type

## Task

> During the migration process, the Nautilus DevOps team deployed multiple EC2 instances across various regions. They are actively monitoring resource usage and making adjustments to optimize performance. Upon identifying an underutilized EC2 instance, they decided to change its instance type to better align with their needs. However, recent security restrictions have revoked AWS console access. Nevertheless, the team retains CLI access via the aws-client host (the landing host for this lab), allowing them to manage AWS resources effectively.
>
> 1. Change the instance type from `t2.micro` to `t2.nano` for `devops-ec2` instance through aws-cli only.
> 2. Make sure the ec2 instance `devops-ec2` is in running state after the change.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-attribute.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/stop-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/start-instances.html

## Steps

```bash
# Get Instance ID.
aws ec2 describe-instances
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "Reservations": [
        {
            "Groups": [],
            "Instances": [
                {
                    "AmiLaunchIndex": 0,
                    "ImageId": "ami-0c101f26f147fa7fd",
                    "InstanceId": "i-0770f3fc4e854e53b",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-08-08T05:09:25.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1b",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-45-221.ec2.internal",
                    "PrivateIpAddress": "172.31.45.221",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-54-158-123-139.compute-1.amazonaws.com",
                    "PublicIpAddress": "54.158.123.139",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-09770965535dd663c",
                    "VpcId": "vpc-0b42c438ba5317405",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-08-08T05:09:25.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-075723074018986d4"
                            }
                        }
                    ],
                    "ClientToken": "40925e30-aae9-40c1-8727-ce7f4da1db11",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-54-158-123-139.compute-1.amazonaws.com",
                                "PublicIp": "54.158.123.139"
                            },
                            "Attachment": {
                                "AttachTime": "2024-08-08T05:09:25.000Z",
                                "AttachmentId": "eni-attach-0bc5c3b5bc1cc8579",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07ab692f5eac98a82"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "0e:5d:4c:17:f8:17",
                            "NetworkInterfaceId": "eni-029cd30f52067c358",
                            "OwnerId": "471112652353",
                            "PrivateDnsName": "ip-172-31-45-221.ec2.internal",
                            "PrivateIpAddress": "172.31.45.221",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-54-158-123-139.compute-1.amazonaws.com",
                                        "PublicIp": "54.158.123.139"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-45-221.ec2.internal",
                                    "PrivateIpAddress": "172.31.45.221"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-09770965535dd663c",
                            "VpcId": "vpc-0b42c438ba5317405",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07ab692f5eac98a82"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Tags": [
                        {
                            "Key": "Name",
                            "Value": "devops-ec2"
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
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpTokens": "required",
                        "HttpPutResponseHopLimit": 2,
                        "HttpEndpoint": "enabled",
                        "HttpProtocolIpv6": "disabled",
                        "InstanceMetadataTags": "disabled"
                    },
                    "EnclaveOptions": {
                        "Enabled": false
                    },
                    "BootMode": "uefi-preferred",
                    "PlatformDetails": "Linux/UNIX",
                    "UsageOperation": "RunInstances",
                    "UsageOperationUpdateTime": "2024-08-08T05:09:25.000Z",
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
            "OwnerId": "471112652353",
            "ReservationId": "r-0f614a8dbea4f368e"
        }
    ]
}
```

</details>

```bash
# Stop EC2.
aws ec2 stop-instances --instance-id i-0770f3fc4e854e53b
```

```json
{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-0770f3fc4e854e53b",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

Wait for the instance to stop.

```bash
# Check Instance status.
 aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": []
}
```

There are no status\` so it is stopped.

```bash
# Change the EC2 type.
aws ec2 modify-instance-attribute \
    --instance-id i-0770f3fc4e854e53b \
    --instance-type "{\"Value\": \"t2.nano\"}"

# Start the EC2 instance.
aws ec2 start-instances --instance-ids i-0770f3fc4e854e53b
```

```json
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-0770f3fc4e854e53b",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
```

Wait for the instance to start.

```bash
# Check Instance status.
 aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1b",
            "InstanceId": "i-0770f3fc4e854e53b",
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
