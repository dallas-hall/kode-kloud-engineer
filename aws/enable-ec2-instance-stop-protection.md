# AWS EC2 Instance Stop Protection

## Task

> As part of the migration, there were some components added to the AWS account. Team created one of the EC2 instances where they need to make some changes now.
>
> * There is an EC2 instance named `devops-ec2` under `us-east-1 region`, enable the `stop` protection for this instance.

## Research

* AWS stop protection.
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-stop-protection.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-stop-protection.html

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
                    "InstanceId": "i-0f2f7c5691d4a86bf",
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2024-07-19T04:56:15.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1b",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-85-100.ec2.internal",
                    "PrivateIpAddress": "172.31.85.100",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-34-227-17-35.compute-1.amazonaws.com",
                    "PublicIpAddress": "34.227.17.35",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-0a3e67a5351957c5a",
                    "VpcId": "vpc-0237414271ff0d01a",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-07-19T04:56:16.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-03937031853cb16dd"
                            }
                        }
                    ],
                    "ClientToken": "f288a021-f0d8-4b3c-aada-83e081ba8b1a",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-34-227-17-35.compute-1.amazonaws.com",
                                "PublicIp": "34.227.17.35"
                            },
                            "Attachment": {
                                "AttachTime": "2024-07-19T04:56:15.000Z",
                                "AttachmentId": "eni-attach-0665875d8aff200b8",
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
                            "MacAddress": "12:9d:3b:8d:8f:45",
                            "NetworkInterfaceId": "eni-0c7afb8ce49a4ae05",
                            "OwnerId": "381491944432",
                            "PrivateDnsName": "ip-172-31-85-100.ec2.internal",
                            "PrivateIpAddress": "172.31.85.100",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-34-227-17-35.compute-1.amazonaws.com",
                                        "PublicIp": "34.227.17.35"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-85-100.ec2.internal",
                                    "PrivateIpAddress": "172.31.85.100"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-0a3e67a5351957c5a",
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
                    "UsageOperationUpdateTime": "2024-07-19T04:56:15.000Z",
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
            "ReservationId": "r-056ca3e5445716169"
        }
    ]
}
```
</details>


```bash
# Enable stop protection.
aws ec2 modify-instance-attribute \
  --instance-id i-0f2f7c5691d4a86bf \
  --disable-api-stop
```
