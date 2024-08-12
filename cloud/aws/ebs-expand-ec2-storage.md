# AWS Expand EC2 Storage

## Task

> The Nautilus DevOps Team has recently been informed by the Development Team that their EC2 instance is running out of storage space. This instance, crucial for development activities, is named `datacenter-ec2` and currently has an attached volume of 8 GiB. To accommodate the increasing data requirements, the storage needs to be expanded to 12 GiB. This change should ensure that the expanded space is immediately available for use within the instance without disrupting ongoing activities.
>
> 1. Identify Volume: Find the volume attached to the `datacenter-ec2` instance.
> 2. Expand Volume: Increase the volume size from 8 GiB to 12 GiB.
> 3. Reflect Changes: Ensure the root (`/`) partition within the instance reflects the expanded size from 8 GiB to 12 GiB.
> 4. SSH Access: Use the key pair located at `/root/datacenter-keypair.pem` on the aws-client host to SSH into the EC2 instance.

## Research

* AWS EBS
  * https://docs.aws.amazon.com/ebs/latest/userguide/requesting-ebs-volume-modifications.html
  * https://docs.aws.amazon.com/ebs/latest/userguide/recognize-expanded-volume-linux.html
  * https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/modify-volume.html
* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-to-linux-instance.html

## Steps

Follow the steps at https://docs.aws.amazon.com/ebs/latest/userguide/requesting-ebs-volume-modifications.html and https://docs.aws.amazon.com/ebs/latest/userguide/recognize-expanded-volume-linux.html and https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-to-linux-instance.html

```bash
# Get EC2 details
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
                    "InstanceId": "i-0c8491f5cdc6249b9",
                    "InstanceType": "t2.micro",
                    "KeyName": "datacenter-keypair",
                    "LaunchTime": "2024-08-12T06:31:59.000Z",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "Placement": {
                        "AvailabilityZone": "us-east-1b",
                        "GroupName": "",
                        "Tenancy": "default"
                    },
                    "PrivateDnsName": "ip-172-31-91-36.ec2.internal",
                    "PrivateIpAddress": "172.31.91.36",
                    "ProductCodes": [],
                    "PublicDnsName": "ec2-3-82-154-206.compute-1.amazonaws.com",
                    "PublicIpAddress": "3.82.154.206",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "StateTransitionReason": "",
                    "SubnetId": "subnet-09d45a5c9df0128e0",
                    "VpcId": "vpc-04b369003c4de6d11",
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "AttachTime": "2024-08-12T06:31:59.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-020d4deda3ed557ed"
                            }
                        }
                    ],
                    "ClientToken": "5b3f3c4d-2473-49be-ad71-42881bbb8660",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-3-82-154-206.compute-1.amazonaws.com",
                                "PublicIp": "3.82.154.206"
                            },
                            "Attachment": {
                                "AttachTime": "2024-08-12T06:31:59.000Z",
                                "AttachmentId": "eni-attach-0057183ec262689f4",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-05a1bf9fe7738fdb5"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "12:0f:49:07:ef:2b",
                            "NetworkInterfaceId": "eni-018567f243335287f",
                            "OwnerId": "339712707627",
                            "PrivateDnsName": "ip-172-31-91-36.ec2.internal",
                            "PrivateIpAddress": "172.31.91.36",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-3-82-154-206.compute-1.amazonaws.com",
                                        "PublicIp": "3.82.154.206"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-91-36.ec2.internal",
                                    "PrivateIpAddress": "172.31.91.36"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-09d45a5c9df0128e0",
                            "VpcId": "vpc-04b369003c4de6d11",
                            "InterfaceType": "interface"
                        }
                    ],
                    "RootDeviceName": "/dev/xvda",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-05a1bf9fe7738fdb5"
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
                    "UsageOperationUpdateTime": "2024-08-12T06:31:59.000Z",
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
            "OwnerId": "339712707627",
            "ReservationId": "r-04acb7aab255a90ad"
        }
    ]
}
```

</details>

```bash
# Get volume ID
aws ec2 describe-volumes
```

```json
{
    "Volumes": [
        {
            "Attachments": [
                {
                    "AttachTime": "2024-08-12T06:31:59.000Z",
                    "Device": "/dev/xvda",
                    "InstanceId": "i-0c8491f5cdc6249b9",
                    "State": "attached",
                    "VolumeId": "vol-020d4deda3ed557ed",
                    "DeleteOnTermination": true
                }
            ],
            "AvailabilityZone": "us-east-1b",
            "CreateTime": "2024-08-12T06:31:59.822Z",
            "Encrypted": false,
            "Size": 8,
            "SnapshotId": "snap-04819fa0d2620618b",
            "State": "in-use",
            "VolumeId": "vol-020d4deda3ed557ed",
            "Iops": 3000,
            "VolumeType": "gp3",
            "MultiAttachEnabled": false,
            "Throughput": 125
        }
    ]
}
```

```bash
# Modify Volume
aws ec2 modify-volume --volume-id vol-020d4deda3ed557ed \
  --size 12
```

```json
{
    "VolumeModification": {
        "VolumeId": "vol-020d4deda3ed557ed",
        "ModificationState": "modifying",
        "TargetSize": 12,
        "TargetIops": 3000,
        "TargetVolumeType": "gp3",
        "TargetThroughput": 125,
        "TargetMultiAttachEnabled": false,
        "OriginalSize": 8,
        "OriginalIops": 3000,
        "OriginalVolumeType": "gp3",
        "OriginalThroughput": 125,
        "OriginalMultiAttachEnabled": false,
        "Progress": 0,
        "StartTime": "2024-08-12T06:44:07.000Z"
    }
}
```

```bash
# Connect to EC2
ssh -i /root/datacenter-keypair.pem ec2-user@ec2-3-82-154-206.compute-1.amazonaws.com

# Become root
sudo -i

# Checks partition sizes.
lsblk
```

```
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0  12G  0 disk
├─xvda1   202:1    0   8G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
```

We can see the `/` partition is still 8 GiB and needs to be expanded.

```bash
# Expand partition.
growpart /dev/xvda 1
```

```
CHANGED: partition=1 start=24576 old: size=16752607 end=16777183 new: size=25141215 end=25165791
```

```bash
# Checks partition sizes.
lsblk
```

```
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0  12G  0 disk
├─xvda1   202:1    0  12G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
```

We can see the `/` partition is 12 GiB now. But the file system will need expanding too.

```bash
# Check file system.
df -hT
```

```
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs          tmpfs     475M     0  475M   0% /dev/shm
tmpfs          tmpfs     190M  2.9M  188M   2% /run
/dev/xvda1     xfs       8.0G  1.6G  6.5G  19% /
tmpfs          tmpfs     475M     0  475M   0% /tmp
/dev/xvda128   vfat       10M  1.3M  8.7M  13% /boot/efi
tmpfs          tmpfs      95M     0   95M   0% /run/user/1000
```

We can see the `/` file system is still 8 GiB and needs to be expanded.

```bash
# Expand the file system.
xfs_growfs -d /
```

```
meta-data=/dev/xvda1             isize=512    agcount=2, agsize=1047040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2094075, imaxpct=25
         =                       sunit=128    swidth=128 blks
naming   =version 2              bsize=16384  ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=4 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2094075 to 3142651
```

```bash
# Check file system.
df -hT
```

```
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs          tmpfs     475M     0  475M   0% /dev/shm
tmpfs          tmpfs     190M  2.9M  188M   2% /run
/dev/xvda1     xfs        12G  1.6G   11G  13% /
tmpfs          tmpfs     475M     0  475M   0% /tmp
/dev/xvda128   vfat       10M  1.3M  8.7M  13% /boot/efi
tmpfs          tmpfs      95M     0   95M   0% /run/user/1000
```

We can see the `/` file system is 12 GiB now.
