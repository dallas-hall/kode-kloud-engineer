# AWS Delete RDS Instance

## Task

> The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their AWS account. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their AWS environment.
>
> Delete the RDS instance named `datacenter-rds`, present in `us-east-1` region. Before submitting this task make sure that RDS instance is properly deleted/removed.

## Research

* AWS RDS
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html
  * https://docs.aws.amazon.com/cli/latest/reference/rds/modify-db-instance.html#examples

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html

```bash
# Disable RDS DB Instance protection.
aws rds modify-db-instance \
    --db-instance-identifier datacenter-rds \
    --no-deletion-protection \
    --apply-immediately
```
<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "datacenter-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "available",
        "MasterUsername": "datacenter_admin",
        "Endpoint": {
            "Address": "datacenter-rds.c72g04y60kys.us-east-1.rds.amazonaws.com",
            "Port": 3306,
            "HostedZoneId": "Z2R2ITUGPM61AM"
        },
        "AllocatedStorage": 5,
        "InstanceCreateTime": "2024-08-07T02:45:09.055Z",
        "PreferredBackupWindow": "09:23-09:53",
        "BackupRetentionPeriod": 1,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-02ab6c9a15d9a92c9",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.0",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "us-east-1d",
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-0760e0fb2fd00489f",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-0968f8dfe339814d7",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1f"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0a3dd7f0a3befa776",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1c"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-00fa074e3de806672",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1e"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0342356d34483b052",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1b"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-06395305d28f3310d",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1a"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0ac192ec0a0eb418c",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1d"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "sat:03:12-sat:03:42",
        "PendingModifiedValues": {},
        "LatestRestorableTime": "2024-08-07T02:46:17.726Z",
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
        "DbiResourceId": "db-ZOMB7D3KNRZKFQK3AF53EZLNXM",
        "CACertificateIdentifier": "rds-ca-rsa2048-g1",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:us-east-1:851725646816:db:datacenter-rds",
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
            "ValidTill": "2025-08-07T02:43:50Z"
        },
        "DedicatedLogVolume": false,
        "EngineLifecycleSupport": "open-source-rds-extended-support"
    }
}
```

</details>

```bash
# Delete RDS DB instance.
aws rds delete-db-instance \
    --db-instance-identifier datacenter-rds \
    --final-db-snapshot-identifier datacenter-rds-final-snapshot \
    --delete-automated-backups
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "datacenter-rds",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "deleting",
        "MasterUsername": "datacenter_admin",
        "Endpoint": {
            "Address": "datacenter-rds.c72g04y60kys.us-east-1.rds.amazonaws.com",
            "Port": 3306,
            "HostedZoneId": "Z2R2ITUGPM61AM"
        },
        "AllocatedStorage": 5,
        "InstanceCreateTime": "2024-08-07T02:45:09.055Z",
        "PreferredBackupWindow": "09:23-09:53",
        "BackupRetentionPeriod": 1,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-02ab6c9a15d9a92c9",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.0",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "us-east-1d",
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-0760e0fb2fd00489f",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-0968f8dfe339814d7",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1f"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0a3dd7f0a3befa776",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1c"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-00fa074e3de806672",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1e"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0342356d34483b052",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1b"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-06395305d28f3310d",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1a"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0ac192ec0a0eb418c",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1d"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "sat:03:12-sat:03:42",
        "PendingModifiedValues": {},
        "LatestRestorableTime": "2024-08-07T02:46:17.726Z",
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
        "DbiResourceId": "db-ZOMB7D3KNRZKFQK3AF53EZLNXM",
        "CACertificateIdentifier": "",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:us-east-1:851725646816:db:datacenter-rds",
        "IAMDatabaseAuthenticationEnabled": false,
        "PerformanceInsightsEnabled": false,
        "DeletionProtection": false,
        "AssociatedRoles": [],
        "TagList": [],
        "CustomerOwnedIpEnabled": false,
        "BackupTarget": "region",
        "NetworkType": "IPV4",
        "StorageThroughput": 0,
        "DedicatedLogVolume": false,
        "EngineLifecycleSupport": "open-source-rds-extended-support"
    }
}
```

</details>

```bash
# Check RDS DB instance.
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]' --output table
```

```
--------------------------------
|      DescribeDBInstances     |
+-----------------+------------+
|  datacenter-rds |  deleting  |
+-----------------+------------+
```

Wait for the final snapshot to be created and check if the RDS DB Instance is deleted.

```bash
# Check RDS DB instance.
aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]' --output table
```

There are no results as the only RDS DB instance was deleted.
