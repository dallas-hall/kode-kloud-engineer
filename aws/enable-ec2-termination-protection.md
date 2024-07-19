# AWS Enable EC2 Instance Termination Protection

## Task

> As part of the migration, there were some components created under the AWS account. The Nautilus DevOps team created one EC2 instance where they forgot to enable the termination protection which is needed for this instance.
>
> * An instance named `devops-ec2` already exists in `us-east-1` region. Enable `termination protection` for the same.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* AWS termination protection
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_ChangingDisableAPITermination.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-attribute.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_ChangingDisableAPITermination.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-attribute.html

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
                    "InstanceId": "i-0d0d344b1c73dedbe",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-07-19T05:04:55.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1a",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-87-232.ec2.internal",
                    "PrivateIpAddress": "172.31.87.232",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-107-22-128-187.compute-1.amazonaws.com",
                    "PublicIpAddress": "107.22.128.187",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-00870951fe50d44b2",
                    "VpcId": "vpc-09e8b3d4d9585fd3e",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-07-19T05:04:56.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-0e340e59419b91242"
                            }
                        }
                    ],
                    "ClientToken": "5e1bb81b-93b0-419b-9ba7-549de748cbbb",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-107-22-128-187.compute-1.amazonaws.com",
                                "PublicIp": "107.22.128.187"
                            },
                            "Attachment": {
                                "AttachTime": "2024-07-19T05:04:55.000Z",
                                "AttachmentId": "eni-attach-077ffcd0b45fa91a1",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-09ee57d37cf7db031"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "12:d6:21:f3:81:55",
                            "NetworkInterfaceId": "eni-079a5da4053e2215b",
                            "OwnerId": "975049905218",
                            "PrivateDnsName": "ip-172-31-87-232.ec2.internal",
                            "PrivateIpAddress": "172.31.87.232",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-107-22-128-187.compute-1.amazonaws.com",
                                        "PublicIp": "107.22.128.187"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-87-232.ec2.internal",
                                    "PrivateIpAddress": "172.31.87.232"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-00870951fe50d44b2",
                            "VpcId": "vpc-09e8b3d4d9585fd3e",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-09ee57d37cf7db031"
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
                    "UsageOperationUpdateTime": "2024-07-19T05:04:55.000Z",
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
            "OwnerId": "975049905218",
            "ReservationId": "r-0d40fd2a4870e5a9b"
        }
    ]
}
```

</details>

```bash
# Enable instance stop protection
aws ec2 modify-instance-attribute \
    --instance-id i-0d0d344b1c73dedbe \
    --disable-api-termination
```
