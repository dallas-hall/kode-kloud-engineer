# AWS RDS Instance Database

## Task

> The Nautilus DevOps team has recently deployed an RDS instance in the `us-east-1` region to serve as a production database. To ensure the instance is equipped with the latest features and security enhancements, the team has identified that the instance is currently running MySQL version `8.0.32`. In light of this, the team is eager to promptly upgrade the database engine version to the latest available version.
>
> RDS instance name is `xfusion-rds`, upgrade the RDS MySQL engine from `v8.0.32`to `v8.0.36`. Also make sure the changes are effective immediately and RDS instance is in Available state.

## Research

* AWS RDS
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Upgrading.html
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.MySQL.html
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/accessing-monitoring.html
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.MySQL.html and https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Upgrading.html

```bash
# Get valid upgradeable RDS DB instances.
aws rds describe-db-engine-versions \
  --engine mysql \
  --engine-version 8.0.32 \
  --query "DBEngineVersions[*].ValidUpgradeTarget[*].{EngineVersion:EngineVersion}" --output text
```

```
8.0.33
8.0.34
8.0.35
8.0.36
8.0.37
```

```bash
# Immediately upgrade RDS DB instance. Do not wait for a maintenance window.
aws rds modify-db-instance \
    --db-instance-identifier xfusion-rds \
    --engine-version 8.0.36 \
    --apply-immediately
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "xfusion-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "available",
        "MasterUsername": "xfusion_admin",
        "Endpoint": {
            "Address": "xfusion-rds.cduoissci144.us-east-1.rds.amazonaws.com",
            "Port": 3306,
            "HostedZoneId": "Z2R2ITUGPM61AM"
        },
        "AllocatedStorage": 5,
        "InstanceCreateTime": "2024-08-07T01:05:57.327Z",
        "PreferredBackupWindow": "10:14-10:44",
        "BackupRetentionPeriod": 1,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-03d6a7252ce01ea28",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.0",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "us-east-1c",
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-00d583f8cfead8589",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-05d4e8a1ee7791154",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1e"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0b40fcd4cdfd8ea86",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1d"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0fd74c3da994a4069",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1c"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-03b7bf0b9f6386511",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1b"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-096124fc34e11652c",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1f"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0b5c7f846940dafcf",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1a"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "sun:04:02-sun:04:32",
        "PendingModifiedValues": {
            "EngineVersion": "8.0.36"
        },
        "LatestRestorableTime": "2024-08-07T01:20:00Z",
        "MultiAZ": false,
        "EngineVersion": "8.0.32",
        "AutoMinorVersionUpgrade": true,
        "ReadReplicaDBInstanceIdentifiers": [],
        "LicenseModel": "general-public-license",
        "OptionGroupMemberships": [
            {
                "OptionGroupName": "default:mysql-8-0",
                "Status": "in-sync"
            }
        ],
        "PubliclyAccessible": true,
        "StorageType": "gp2",
        "DbInstancePort": 0,
        "StorageEncrypted": false,
        "DbiResourceId": "db-3XIOIPD43B5UEETONINIDVWACI",
        "CACertificateIdentifier": "rds-ca-rsa2048-g1",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:us-east-1:058264522852:db:xfusion-rds",
        "IAMDatabaseAuthenticationEnabled": false,
        "PerformanceInsightsEnabled": false,
        "DeletionProtection": false,
        "AssociatedRoles": [],
        "TagList": [],
        "CustomerOwnedIpEnabled": false,
        "BackupTarget": "region",
        "NetworkType": "IPV4",
        "StorageThroughput": 0,
        "CertificateDetails": {
            "CAIdentifier": "rds-ca-rsa2048-g1",
            "ValidTill": "2025-08-07T01:04:38Z"
        },
        "DedicatedLogVolume": false,
        "EngineLifecycleSupport": "open-source-rds-extended-support"
    }
}
```

</details>


```bash
# View RDS instance status`
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]' --output table
```

```
------------------------------
|     DescribeDBInstances    |
+--------------+-------------+
|  xfusion-rds |  upgrading  |
+--------------+-------------+
```

Wait a few minutes for the changes to be applied.

```bash
# View RDS instance status`
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]' --output table
```

```
------------------------------
|     DescribeDBInstances    |
+--------------+-------------+
|  xfusion-rds |  available  |
+--------------+-------------+
```


```bash
# View RDS instance version.
aws rds describe-db-instances --db-instance-identifier xfusion-rds | jq .DBInstances[0].EngineVersion
```

```
"8.0.36"
```
