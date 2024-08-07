# Change EC2 Instance

## Task

> During the migration process, the Nautilus DevOps team created several EC2 instances in different regions. They are currently in the process of identifying the correct resources and utilization and are making continuous changes to ensure optimal resource utilization. Recently, they discovered that one of the EC2 instances was underutilized, prompting them to decide to change the instance type. Please make sure the Status check is completed (if its still in Initializing state) before making any changes to the instance.
>
> 1. Change the instance type from `t2.micro` to `t2.nano` for `xfusion-ec2` instance.
> 2. Make sure the ec2 instance `xfusion-ec2` is in running state after the change.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html#change-instance-type-of-ebs-backed-instance

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html and https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html#change-instance-type-of-ebs-backed-instance

**NOTE:** It takes time for the instance to stop and to be able to change the instance type. Be patient!
