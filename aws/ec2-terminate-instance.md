# AWS Terminate EC2 Instance

## Task

> During the migration process, several resources were created under the AWS account. Later on, some of these resources became obsolete as alternative solutions were implemented. Similarly, there is an instance that needs to be deleted as it is no longer in use.
>
> 1. Delete the ec2 instance named `devops-ec2` present in `us-east-1` region.
> 2. Before submitting your task, make sure instance is in `terminated` state.

## Research

* AWS terminate EC2.
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html

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
                    "InstanceId": "i-0abcf471b347795d1",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-07-23T08:38:19.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1c",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-90-7.ec2.internal",
                    "PrivateIpAddress": "172.31.90.7",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-18-212-17-206.compute-1.amazonaws.com",
                    "PublicIpAddress": "18.212.17.206",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-0177595d14ba0be11",
                    "VpcId": "vpc-052ab32ec298e35ba",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-07-23T08:38:20.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-0210da6f34ded2833"
                            }
                        }
                    ],
                    "ClientToken": "873c3a78-078b-48f8-95e7-61ab1b39acb5",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-18-212-17-206.compute-1.amazonaws.com",
                                "PublicIp": "18.212.17.206"
                            },
                            "Attachment": {
                                "AttachTime": "2024-07-23T08:38:19.000Z",
                                "AttachmentId": "eni-attach-0a3d36b46821e455b",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-00d9fe44c790dc1eb"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "12:7b:f4:04:e2:ff",
                            "NetworkInterfaceId": "eni-04f1c0fe3950a1066",
                            "OwnerId": "211125513710",
                            "PrivateDnsName": "ip-172-31-90-7.ec2.internal",
                            "PrivateIpAddress": "172.31.90.7",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-18-212-17-206.compute-1.amazonaws.com",
                                        "PublicIp": "18.212.17.206"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-90-7.ec2.internal",
                                    "PrivateIpAddress": "172.31.90.7"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-0177595d14ba0be11",
                            "VpcId": "vpc-052ab32ec298e35ba",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-00d9fe44c790dc1eb"
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
                    "UsageOperationUpdateTime": "2024-07-23T08:38:19.000Z",
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
            "OwnerId": "211125513710",
            "ReservationId": "r-00295c1c2e4e49728"
        }
    ]
}
```

</details>

```bash
# Terminate EC2.
aws ec2 terminate-instances --instance-ids i-0abcf471b347795d1
```

```json
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-0abcf471b347795d1",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```
