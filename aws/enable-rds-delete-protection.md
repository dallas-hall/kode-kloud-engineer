# AWS Enable RDS Instance Deletion Protection

## Task

> The Nautilus DevOps team has recently deployed an RDS instance in the us-east-1 region. As this instance will be utilized for a production database, ensuring its security and protection is paramount. Upon inspection, it was discovered that delete/termination protection is not currently enabled for this instance. Consequently, the team aims to promptly enable this protection feature to prevent accidental deletion or termination of the instance.
>
> * RDS instance name is `datacenter-rds`, also make sure changes are effective immediately.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html
  * https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html#examples

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html or https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html#examples

```bash
# Enable RDS deletion protection
aws rds modify-db-instance \
    --db-instance-identifier datacenter-rds \
    --deletion-protection
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "datacenter-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "backing-up",
        "MasterUsername": "datacenter_admin",
        "Endpoint": {
            "Address": "datacenter-rds.cj6w02cuehgf.us-east-1.rds.amazonaws.com",
            "Port": 3306,
            "HostedZoneId": "Z2R2ITUGPM61AM"
        },
        "AllocatedStorage": 5,
        "InstanceCreateTime": "2024-08-05T04:09:49.257Z",
        "PreferredBackupWindow": "07:23-07:53",
        "BackupRetentionPeriod": 1,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-09e0413e78bc75f2c",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.0",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "us-east-1a",
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-0b52573c291ec299f",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-03ea5abe70cae5e93",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1b"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0a01d60e9bb9a299f",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1f"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-06af749f91f7b5588",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1d"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0a40b885a564c31ae",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1c"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-08272c883567a658d",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1e"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-05e91819635db5f7a",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1a"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "sat:09:59-sat:10:29",
        "PendingModifiedValues": {},
        "MultiAZ": false,
        "EngineVersion": "8.0.36",
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
        "DbiResourceId": "db-BCWYSDCLXN26DCBZZXW45OJL6A",
        "CACertificateIdentifier": "rds-ca-rsa2048-g1",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:us-east-1:975049900668:db:datacenter-rds",
        "IAMDatabaseAuthenticationEnabled": false,
        "PerformanceInsightsEnabled": false,
        "DeletionProtection": true,
        "AssociatedRoles": [],
        "TagList": [],
        "CustomerOwnedIpEnabled": false,
        "BackupTarget": "region",
        "NetworkType": "IPV4",
        "StorageThroughput": 0,
        "CertificateDetails": {
            "CAIdentifier": "rds-ca-rsa2048-g1",
            "ValidTill": "2025-08-05T04:08:29Z"
        },
        "DedicatedLogVolume": false,
        "EngineLifecycleSupport": "open-source-rds-extended-support"
    }
}
```

</details>
