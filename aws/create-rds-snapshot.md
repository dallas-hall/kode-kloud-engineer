# AWS RDS Instance Snapshot

## Task

> When it comes to managing databases, implementing backup and restore processes is a crucial task. It's essential to ensure that a backup process is in place to safeguard data and that restoration processes are available for disaster recovery scenarios. The Nautilus team has outlined the following requirements in this regard:
>
> * One RDS instance named `nautilus-rds` is already running in `us-east-1` region, take a snapshot of this RDS instance and name it as `nautilus-rds-snapshot`.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html

```bash
# Create RDS DB instance snapshot.
aws rds create-db-snapshot \
    --db-instance-identifier nautilus-rds \
    --db-snapshot-identifier nautilus-rds-snapshot
```

```json
{
    "DBSnapshot": {
        "DBSnapshotIdentifier": "nautilus-rds-snapshot",
        "DBInstanceIdentifier": "nautilus-rds",
        "Engine": "mysql",
        "AllocatedStorage": 5,
        "Status": "creating",
        "Port": 3306,
        "AvailabilityZone": "us-east-1d",
        "VpcId": "vpc-0e2adf2fa755f7702",
        "InstanceCreateTime": "2024-08-05T03:49:48.428Z",
        "MasterUsername": "nautilus_admin",
        "EngineVersion": "8.0.36",
        "LicenseModel": "general-public-license",
        "SnapshotType": "manual",
        "OptionGroupName": "default:mysql-8-0",
        "PercentProgress": 0,
        "StorageType": "gp2",
        "Encrypted": false,
        "DBSnapshotArn": "arn:aws:rds:us-east-1:654654139377:snapshot:nautilus-rds-snapshot",
        "IAMDatabaseAuthenticationEnabled": false,
        "ProcessorFeatures": [],
        "DbiResourceId": "db-2XBN7ZTC2BBOONV4H5XIMKXCFA",
        "TagList": [],
        "SnapshotTarget": "region",
        "StorageThroughput": 0,
        "DedicatedLogVolume": false
    }
}
```
