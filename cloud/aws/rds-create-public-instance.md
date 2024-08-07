# AWS Public RDS Instance

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognising the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. Recently they started working on creating and configuring some database instances on AWS.
>
> For this task, create one publicly accessible RDS instance using a free tier template along with the following details:
> 1. The name of the RDS instance must be `nautilus-rds`.
> 2. The engine type must be `MySQL v8.0.36`, further it must be a `db.t3.micro` type instance.
> 3. The master username must be `nautilus_admin` and set some appropriate password.
> 4. RDS storage type must be `gp2` and storage size must be `5GiB`.
> 5. Keep the rest of the configurations as `default`. Finally, make sure the instance is in `available` state before submitting this task.

## Research

* AWS RDS
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html
  * https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html
