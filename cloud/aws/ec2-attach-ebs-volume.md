# AWS Attach EBS Volume Interface to EC2 Instance

## Task

> The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
>
> An instance named `datacenter-ec2` and a volume named `datacenter-volume` already exists in `us-east-1` region. Attach the `datacenter-volume` volume to the `datacenter-ec2` instance, make sure to set the device name to `/dev/sdb` while attaching the volume.

## Research

* AWS attach EBS volume.
  * https://docs.aws.amazon.com/ebs/latest/userguide/ebs-attaching-volume.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-volume.html

## Steps

Follow the steps at https://docs.aws.amazon.com/ebs/latest/userguide/ebs-attaching-volume.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-volume.html

```bash
# Get instance ID
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
                    "InstanceId": "i-0b5b5771f243bfa50",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-07-22T01:13:56.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1a",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-27-106.ec2.internal",
                    "PrivateIpAddress": "172.31.27.106",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-3-84-121-192.compute-1.amazonaws.com",
                    "PublicIpAddress": "3.84.121.192",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-0d9fe3271a134812b",
                    "VpcId": "vpc-0125ece593b2c2b05",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-07-22T01:13:57.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-0189a6f5b0f3038f4"
                            }
                        }
                    ],
                    "ClientToken": "f7f27fd3-46ca-45c6-be54-643f742b889e",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-3-84-121-192.compute-1.amazonaws.com",
                                "PublicIp": "3.84.121.192"
                            },
                            "Attachment": {
                                "AttachTime": "2024-07-22T01:13:56.000Z",
                                "AttachmentId": "eni-attach-0373716474e7d3b86",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-0bc41a14dfb2a291c"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "0a:ff:c7:34:fd:f5",
                            "NetworkInterfaceId": "eni-0af5a4fa1182765c7",
                            "OwnerId": "637423209198",
                            "PrivateDnsName": "ip-172-31-27-106.ec2.internal",
                            "PrivateIpAddress": "172.31.27.106",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-3-84-121-192.compute-1.amazonaws.com",
                                        "PublicIp": "3.84.121.192"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-27-106.ec2.internal",
                                    "PrivateIpAddress": "172.31.27.106"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-0d9fe3271a134812b",
                            "VpcId": "vpc-0125ece593b2c2b05",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-0bc41a14dfb2a291c"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Tags": [
                        {
                            "Key": "Name",
                            "Value": "datacenter-ec2"
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
                    "UsageOperationUpdateTime": "2024-07-22T01:13:56.000Z",
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
            "OwnerId": "637423209198",
            "ReservationId": "r-00057e6e616cc923b"
        }
    ]
}
```

</details>


```bash
# Get volume ID
aws ec2 aws ec2 describe-volumes
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "Volumes": [
        {
            "Attachments": [
                {
                    "AttachTime": "2024-07-22T01:13:57.000Z",
                    "Device": "/dev/xvda",
                    "InstanceId": "i-0b5b5771f243bfa50",
                    "State": "attached",
                    "VolumeId": "vol-0189a6f5b0f3038f4",
                    "DeleteOnTermination": true
                }
            ],
            "AvailabilityZone": "us-east-1a",
            "CreateTime": "2024-07-22T01:13:57.102Z",
            "Encrypted": false,
            "Size": 8,
            "SnapshotId": "snap-04819fa0d2620618b",
            "State": "in-use",
            "VolumeId": "vol-0189a6f5b0f3038f4",
            "Iops": 3000,
            "VolumeType": "gp3",
            "MultiAttachEnabled": false,
            "Throughput": 125
        },
        {
            "Attachments": [],
            "AvailabilityZone": "us-east-1a",
            "CreateTime": "2024-07-22T01:13:57.471Z",
            "Encrypted": false,
            "Size": 5,
            "SnapshotId": "",
            "State": "available",
            "VolumeId": "vol-046fb2e82f2f2bb3c",
            "Iops": 100,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "datacenter-volume"
                }
            ],
            "VolumeType": "gp2",
            "MultiAttachEnabled": false
        }
    ]
}
```

</details>


```bash
# Attach ebs volumne to ec2 instance.
aws ec2 attach-volume \
--volume-id vol-046fb2e82f2f2bb3c \
--instance-id i-0b5b5771f243bfa50 \
--device /dev/sdb
```

```json
{
    "AttachTime": "2024-07-22T01:21:46.748Z",
    "Device": "/dev/sdb",
    "InstanceId": "i-0b5b5771f243bfa50",
    "State": "attaching",
    "VolumeId": "vol-046fb2e82f2f2bb3c"
}
```
