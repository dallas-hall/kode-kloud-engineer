# AWS Create EBS Volume Snapshot

## Task

> The Nautilus DevOps team has some volumes in different regions in their AWS account. They are going to setup some automated backups so that all important data can be backed up on regular basis. For now they shared some requirements to take a snapshot of one of the volumes they have.
>
> Create a snapshot of an existing volume named `nautilus-vol` in `us-east-1` region.
> 1. The name of the snapshot must be `nautilus-vol-ss`.
> 2. The description must be `nautilus Snapshot`.
> 3. Make sure the snapshot status is `completed` before submitting the task.

## Research

* AWS Create EBS Volume Snapshot.
  * https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-snapshot.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-snapshot.html

## Steps

Follow the steps at https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-snapshot.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/create-snapshot.html

```bash
# Get volume ID
aws ec2 describe-volumes
```

```json
{
    "Volumes": [
        {
            "Attachments": [],
            "AvailabilityZone": "us-east-1a",
            "CreateTime": "2024-07-23T08:46:40.337Z",
            "Encrypted": false,
            "Size": 5,
            "SnapshotId": "",
            "State": "available",
            "VolumeId": "vol-0a98f9a16055fe30b",
            "Iops": 100,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "nautilus-vol"
                }
            ],
            "VolumeType": "gp2",
            "MultiAttachEnabled": false
        }
    ]
}
```

```bash
# Create EBS Volume Snapshot
aws ec2 create-snapshot \
--volume-id vol-0a98f9a16055fe30b \
--description 'nautilus Snapshot' \
--tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=nautilus-vol-ss}]'
```

```json
{
    "Description": "nautilus Snapshot",
    "Encrypted": false,
    "OwnerId": "851725514760",
    "Progress": "",
    "SnapshotId": "snap-030e960b43d1e5b7e",
    "StartTime": "2024-07-23T08:53:01.273Z",
    "State": "pending",
    "VolumeId": "vol-0a98f9a16055fe30b",
    "VolumeSize": 5,
    "Tags": [
        {
            "Key": "Name",
            "Value": "nautilus-vol-ss"
        }
    ]
}
```
