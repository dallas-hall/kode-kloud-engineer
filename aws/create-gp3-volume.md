# AWS Create GP3 Volume

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> Create a volume with the following requirements:
> * Name of the volume should be `devops-volume`.
> * Volume type must be `gp3`.
> * Volume size must be `2 GiB`.
## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* AWS create GP3 Volume
  * https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-volume.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-volume.html

## Steps

Follow the steps at https://docs.aws.amazon.com/ebs/latest/userguide/ebs-creating-volume.html or https://docs.aws.amazon.com/cli/latest/reference/ec2/create-volume.html

```bash
# Create the GP3 Volume.
aws ec2 create-volume \
    --volume-type gp3 \
    --size 2 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=volume,Tags=[{Key=name,Value=devops-volume}]'
```

```json
{
    "AvailabilityZone": "us-east-1a",
    "CreateTime": "2024-07-17T08:37:57.000Z",
    "Encrypted": false,
    "Size": 2,
    "SnapshotId": "",
    "State": "creating",
    "VolumeId": "vol-04eeac277ba234b0a",
    "Iops": 3000,
    "Tags": [
        {
            "Key": "name",
            "Value": "devops-volume"
        }
    ],
    "VolumeType": "gp3",
    "MultiAttachEnabled": false,
    "Throughput": 125
}
```