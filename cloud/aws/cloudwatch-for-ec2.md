# Launch EC2 Instance With CloudWatch Alarm

## Task

> The Nautilus DevOps team has been tasked with setting up an EC2 instance for their application. To ensure the application performs optimally, they also need to create a CloudWatch alarm to monitor the instance's CPU utilization. The alarm should trigger if the CPU utilization exceeds 90% for one consecutive 5-minute period. To send notifications, use the SNS topic named `nautilus-sns-topic` which is already created.
>
> * Launch EC2 Instance: Create an EC2 instance named `nautilus-ec2` using any appropriate Ubuntu AMI.
> * Create CloudWatch Alarm: Create a CloudWatch alarm named `nautilus-alarm` with the following specifications:
>   * Statistic: Average
>   * Metric: CPU Utilization
>   * Threshold: >= 90% for 1 consecutive 5-minute period.
>   * Alarm Actions: Send a notification to nautilus-sns-topic.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
* AWS CloudWatch
  * https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
  * https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GettingSetup.html
  * https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudwatch/index.html
* AWS SNS
  * https://aws.amazon.com/sns/
  * https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sns/index.html

## Steps

For GUI, follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html and https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GettingSetup.html

For CLI, follow the steps at https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html and https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudwatch/put-metric-alarm.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name nautilus-key-pair \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > nautilus-key-pair.pem

# Launch EC2.
aws ec2 run-instances --image-id ami-0cd59ecaf368e5ccf \
  --count 1 \
  --instance-type t2.micro \
  --key-name nautilus-key-pair \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]'
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0cd59ecaf368e5ccf",
            "InstanceId": "i-04e6bd8b0d29442be",
            "InstanceType": "t2.micro",
            "KeyName": "nautilus-key-pair",
            "LaunchTime": "2024-09-19T06:01:46.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-38-156.ec2.internal",
            "PrivateIpAddress": "172.31.38.156",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0f993fbf1439cb9ff",
            "VpcId": "vpc-0136d35f15b9709d0",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "0bd5a22a-2b8f-4230-980b-a4ea37f2c07d",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2024-09-19T06:01:46.000Z",
                        "AttachmentId": "eni-attach-0066a930e39e0ca5e",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-06874fb2c7ba14197"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0e:c0:5a:0a:cc:39",
                    "NetworkInterfaceId": "eni-0520daea34bcc317e",
                    "OwnerId": "992382848249",
                    "PrivateDnsName": "ip-172-31-38-156.ec2.internal",
                    "PrivateIpAddress": "172.31.38.156",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-38-156.ec2.internal",
                            "PrivateIpAddress": "172.31.38.156"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0f993fbf1439cb9ff",
                    "VpcId": "vpc-0136d35f15b9709d0",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-06874fb2c7ba14197"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "nautilus-ec2"
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
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
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
    "OwnerId": "992382848249",
    "ReservationId": "r-03e8b201968d84ddd"
}
```

</details>

Wait a few minutes for it to start up.

```bash
# Check EC2 status.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1d",
            "InstanceId": "i-04e6bd8b0d29442be",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "InstanceStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            },
            "SystemStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "passed"
                    }
                ],
                "Status": "ok"
            }
        }
    ]
}
```

Get SNS ID.

```bash
aws sns list-topics
```

```json
{
    "Topics": [
        {
            "TopicArn": "arn:aws:sns:us-east-1:992382848249:nautilus-sns-topic"
        }
    ]
}
```

Create CloudWatch Alarm.

```bash
aws cloudwatch put-metric-alarm --alarm-name nautilus-alarm \
    --alarm-description "Alarm when CPU >= 90 percent" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --evaluation-periods 1 \
    --threshold 90 \
    --unit Percent \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --dimensions "Name=InstanceId,Value=i-04e6bd8b0d29442be" \
    --alarm-actions arn:aws:sns:us-east-1:992382848249:nautilus-sns-topic
```

View the CloudWatch Alarm.

```bash
aws cloudwatch describe-alarms
```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "MetricAlarms": [
        {
            "AlarmName": "nautilus-alarm",
            "AlarmArn": "arn:aws:cloudwatch:us-east-1:992382848249:alarm:nautilus-alarm",
            "AlarmDescription": "Alarm when CPU >= 90 percent",
            "AlarmConfigurationUpdatedTimestamp": "2024-09-19T06:09:11.090Z",
            "ActionsEnabled": true,
            "OKActions": [],
            "AlarmActions": [
                "arn:aws:sns:us-east-1:992382848249:nautilus-sns-topic"
            ],
            "InsufficientDataActions": [],
            "StateValue": "INSUFFICIENT_DATA",
            "StateReason": "Unchecked: Initial alarm creation",
            "StateUpdatedTimestamp": "2024-09-19T06:09:11.090Z",
            "MetricName": "CPUUtilization",
            "Namespace": "AWS/EC2",
            "Statistic": "Average",
            "Dimensions": [
                {
                    "Name": "InstanceId",
                    "Value": "i-04e6bd8b0d29442be"
                }
            ],
            "Period": 300,
            "Unit": "Percent",
            "EvaluationPeriods": 2,
            "Threshold": 90.0,
            "ComparisonOperator": "GreaterThanOrEqualToThreshold",
            "StateTransitionedTimestamp": "2024-09-19T06:09:11.090Z"
        },
        {
            "AlarmName": "nautilus-alarm-gui",
            "AlarmArn": "arn:aws:cloudwatch:us-east-1:992382848249:alarm:nautilus-alarm-gui",
            "AlarmConfigurationUpdatedTimestamp": "2024-09-19T06:00:31.626Z",
            "ActionsEnabled": true,
            "OKActions": [],
            "AlarmActions": [
                "arn:aws:sns:us-east-1:992382848249:nautilus-sns-topic"
            ],
            "InsufficientDataActions": [],
            "StateValue": "OK",
            "StateReason": "Threshold Crossed: 1 out of the last 1 datapoints [1.0166666666666666 (19/09/24 05:56:00)] was not greater than or equal to the threshold (90.0) (minimum 1 datapoint for ALARM -> OK transition).",
            "StateReasonData": "{\"version\":\"1.0\",\"queryDate\":\"2024-09-19T06:01:18.043+0000\",\"startDate\":\"2024-09-19T05:56:00.000+0000\",\"statistic\":\"Average\",\"period\":300,\"recentDatapoints\":[1.0166666666666666],\"threshold\":90.0,\"evaluatedDatapoints\":[{\"timestamp\":\"2024-09-19T05:56:00.000+0000\",\"sampleCount\":1.0,\"value\":1.0166666666666666}]}",
            "StateUpdatedTimestamp": "2024-09-19T06:01:18.045Z",
            "MetricName": "CPUUtilization",
            "Namespace": "AWS/EC2",
            "Statistic": "Average",
            "Dimensions": [
                {
                    "Name": "InstanceId",
                    "Value": "i-0237f76ff9f9a119e"
                }
            ],
            "Period": 300,
            "EvaluationPeriods": 1,
            "DatapointsToAlarm": 1,
            "Threshold": 90.0,
            "ComparisonOperator": "GreaterThanOrEqualToThreshold",
            "TreatMissingData": "missing",
            "StateTransitionedTimestamp": "2024-09-19T06:01:18.045Z"
        }
    ],
    "CompositeAlarms": []
}
```

</details>

We are done.
