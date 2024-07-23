# AWS Attach Elastic Network Interface to EC2 Instance

## Task

> The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
>
> An instance named `nautilus-ec2`and an elastic network interface named `nautilus-eni` already exists in `us-east-1` region.
> * Attach the `nautilus-eni` network interface to the `nautilus-ec2` instance.
> * Make sure status is `attached` before submitting the task.

## Research

* AWS attach Elastic network interface.
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#attach_eni
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-network-interface.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#attach_eni or https://docs.aws.amazon.com/cli/latest/reference/ec2/attach-network-interface.html

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
                    "InstanceId": "i-02725270729dce83c",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-07-22T01:01:41.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1a",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-85-125.ec2.internal",
                    "PrivateIpAddress": "172.31.85.125",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-52-207-182-166.compute-1.amazonaws.com",
                    "PublicIpAddress": "52.207.182.166",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-007ba0ca8fc3f9407",
                    "VpcId": "vpc-0874e4cc953fe3e59",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-07-22T01:01:41.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-048a83e02baf8708c"
                            }
                        }
                    ],
                    "ClientToken": "ade82f86-aae7-4a79-b6ad-9e070d309958",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-52-207-182-166.compute-1.amazonaws.com",
                                "PublicIp": "52.207.182.166"
                            },
                            "Attachment": {
                                "AttachTime": "2024-07-22T01:01:41.000Z",
                                "AttachmentId": "eni-attach-0d0dd247161b9081b",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-0e94fac69c9f27199"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "12:96:be:e3:68:73",
                            "NetworkInterfaceId": "eni-0df4b8d412e67049e",
                            "OwnerId": "992382436308",
                            "PrivateDnsName": "ip-172-31-85-125.ec2.internal",
                            "PrivateIpAddress": "172.31.85.125",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-52-207-182-166.compute-1.amazonaws.com",
                                        "PublicIp": "52.207.182.166"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-85-125.ec2.internal",
                                    "PrivateIpAddress": "172.31.85.125"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-007ba0ca8fc3f9407",
                            "VpcId": "vpc-0874e4cc953fe3e59",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-0e94fac69c9f27199"
                        }
                    ],
                    "SourceDestCheck": true,
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
                    "UsageOperationUpdateTime": "2024-07-22T01:01:41.000Z",
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
            "OwnerId": "992382436308",
            "ReservationId": "r-0935fbb8fc739ef54"
        }
    ]
}
```

</details>


```bash
# Get network interface ID
aws ec2 aws ec2 describe-network-interfaces
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "NetworkInterfaces": [
        {
            "Association": {
                "IpOwnerId": "amazon",
                "PublicDnsName": "ec2-52-207-182-166.compute-1.amazonaws.com",
                "PublicIp": "52.207.182.166"
            },
            "Attachment": {
                "AttachTime": "2024-07-22T01:01:41.000Z",
                "AttachmentId": "eni-attach-0d0dd247161b9081b",
                "DeleteOnTermination": true,
                "DeviceIndex": 0,
                "NetworkCardIndex": 0,
                "InstanceId": "i-02725270729dce83c",
                "InstanceOwnerId": "992382436308",
                "Status": "attached"
            },
            "AvailabilityZone": "us-east-1a",
            "Description": "",
            "Groups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-0e94fac69c9f27199"
                }
            ],
            "InterfaceType": "interface",
            "Ipv6Addresses": [],
            "MacAddress": "12:96:be:e3:68:73",
            "NetworkInterfaceId": "eni-0df4b8d412e67049e",
            "OwnerId": "992382436308",
            "PrivateDnsName": "ip-172-31-85-125.ec2.internal",
            "PrivateIpAddress": "172.31.85.125",
            "PrivateIpAddresses": [
                {
                    "Association": {
                        "IpOwnerId": "amazon",
                        "PublicDnsName": "ec2-52-207-182-166.compute-1.amazonaws.com",
                        "PublicIp": "52.207.182.166"
                    },
                    "Primary": true,
                    "PrivateDnsName": "ip-172-31-85-125.ec2.internal",
                    "PrivateIpAddress": "172.31.85.125"
                }
            ],
            "RequesterManaged": false,
            "SourceDestCheck": true,
            "Status": "in-use",
            "SubnetId": "subnet-007ba0ca8fc3f9407",
            "TagSet": [],
            "VpcId": "vpc-0874e4cc953fe3e59"
        },
        {
            "AvailabilityZone": "us-east-1a",
            "Description": "nautilus-eni",
            "Groups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-0e94fac69c9f27199"
                }
            ],
            "InterfaceType": "interface",
            "Ipv6Addresses": [],
            "MacAddress": "12:fc:a4:62:89:0b",
            "NetworkInterfaceId": "eni-0eb26eb7783158352",
            "OwnerId": "992382436308",
            "PrivateDnsName": "ip-172-31-85-108.ec2.internal",
            "PrivateIpAddress": "172.31.85.108",
            "PrivateIpAddresses": [
                {
                    "Primary": true,
                    "PrivateDnsName": "ip-172-31-85-108.ec2.internal",
                    "PrivateIpAddress": "172.31.85.108"
                }
            ],
            "RequesterId": "AIDA6ODU2IPKDNTCWVZBF",
            "RequesterManaged": false,
            "SourceDestCheck": true,
            "Status": "available",
            "SubnetId": "subnet-007ba0ca8fc3f9407",
            "TagSet": [
                {
                    "Key": "Name",
                    "Value": "nautilus-eni"
                }
            ],
            "VpcId": "vpc-0874e4cc953fe3e59"
        }
    ]
}
```

</details>


```bash
# Attach network interface to ec2 instance.
aws ec2 attach-network-interface \
    --network-interface-id eni-0eb26eb7783158352 \
    --instance-id i-02725270729dce83c \
    --device-index 1
```

```json
{
    "AttachmentId": "eni-attach-0c4ca550ebdcdb23d",
    "NetworkCardIndex": 0
}
```
