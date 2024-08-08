# Delete EC2 Instance

## Task

> During the migration process, several resources were created under the AWS account. Later on, some of these resources became obsolete as alternative solutions were implemented. Similarly, there is an instance that needs to be deleted as it is no longer in use. Recent security restrictions have revoked AWS console access. Nevertheless, the team retains CLI access via the aws-client host (the landing host for this lab), allowing them to manage AWS resources effectively.
>
> 1. Using AWS CLI, delete an ec2 instance named `datacenter-ec2` present in `us-east-1` region.
> 2. Before submitting your task, make sure instance is in terminated state.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html#terminating-instances
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html

## Steps

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
                    "InstanceId": "i-0176ce2558f70ae16",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-08-08T05:47:42.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1d",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-36-116.ec2.internal",
                    "PrivateIpAddress": "172.31.36.116",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-3-81-115-158.compute-1.amazonaws.com",
                    "PublicIpAddress": "3.81.115.158",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-0b65c40723d1e7ba5",
                    "VpcId": "vpc-0237414271ff0d01a",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-08-08T05:47:43.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-0c7ea8ef0dc411412"
                            }
                        }
                    ],
                    "ClientToken": "b34df97a-9d75-419c-b91b-61c99bbc5029",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-3-81-115-158.compute-1.amazonaws.com",
                                "PublicIp": "3.81.115.158"
                            },
                            "Attachment": {
                                "AttachTime": "2024-08-08T05:47:42.000Z",
                                "AttachmentId": "eni-attach-01f5801d2fe885d96",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-0e190fd026eb2dd31"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "0e:3b:e0:4d:6a:af",
                            "NetworkInterfaceId": "eni-0f7fad957b2b15743",
                            "OwnerId": "381491944432",
                            "PrivateDnsName": "ip-172-31-36-116.ec2.internal",
                            "PrivateIpAddress": "172.31.36.116",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-3-81-115-158.compute-1.amazonaws.com",
                                        "PublicIp": "3.81.115.158"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-36-116.ec2.internal",
                                    "PrivateIpAddress": "172.31.36.116"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-0b65c40723d1e7ba5",
                            "VpcId": "vpc-0237414271ff0d01a",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-0e190fd026eb2dd31"
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
                    "UsageOperationUpdateTime": "2024-08-08T05:47:42.000Z",
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
            "OwnerId": "381491944432",
            "ReservationId": "r-017427509addc4b86"
        }
    ]
}
```

</details>

```bash
# Delete instance.
aws ec2 terminate-instances --instance-ids i-0176ce2558f70ae16
```

```json
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-0176ce2558f70ae16",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```
```bash
# Describe active instances.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": []
}
```

There are none so it is terminated. But lets double check.

```bash
# Describe all instances.
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
                    "InstanceId": "i-0176ce2558f70ae16",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-08-08T05:47:42.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1d",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "",
                    "ProductCodes": [],
                    "PublicDnsName": "",
                    "State": {
                        "Code": 48,
                        "Name": "terminated"
                    },
                    "StateTransitionReason": "User initiated (2024-08-08 05:54:09 GMT)",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [],
                    "ClientToken": "b34df97a-9d75-419c-b91b-61c99bbc5029",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [],
                    "StateReason": {
                        "Code": "Client.UserInitiatedShutdown",
                        "Message": "Client.UserInitiatedShutdown: User initiated shutdown"
                    },
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
                        "State": "pending",
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
                    "UsageOperationUpdateTime": "2024-08-08T05:47:42.000Z",
                    "MaintenanceOptions": {
                        "AutoRecovery": "default"
                    },
                    "CurrentInstanceBootMode": "legacy-bios"
                }
            ],
            "OwnerId": "381491944432",
            "ReservationId": "r-017427509addc4b86"
        }
    ]
}
```

</details>

We can see the status is `terminated`.