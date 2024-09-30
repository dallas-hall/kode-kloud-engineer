# Launch EC2 Instance With nginx

## Task

> The Nautilus DevOps Team is working on setting up a new web server for a critical application. The team lead has requested you to create an EC2 instance that will serve as a web server using Nginx. This instance will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.
>
> As a member of the Nautilus DevOps Team, your task is to create an EC2 instance with the following specifications:
> * Instance Name: The EC2 instance must be named `devops-ec2`.
> * AMI: Use any available Ubuntu AMI to create this instance.
> * User Data Script: Configure the instance to run a user data script during its launch. This script should:
>   * Install the Nginx package.
>   * Start the Nginx service.
> * Security Group: Ensure that the instance allows HTTP traffic on port 80 from the internet.

## Research


* Security Groups:
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-security-groups.html#creating-security-group
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html
* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
  * https://stackoverflow.com/questions/56994488/how-to-get-public-ip-of-an-ec2-instance-from-aws-cli-by-instance-id
* AWS EC2 UserData Script
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-api-cli
  * https://stackoverflow.com/questions/15904095/how-to-check-whether-my-user-data-passing-to-ec2-instance-is-working

## Steps

Follow the steps at https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name devops-keypair \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > devops-keypair.pem

# Create the Security Group.
aws ec2 create-security-group \
--group-name devops-http-sg \
--description "Security group for devops nginx HTTP Servers"
```

```json
{
    "GroupId": "sg-06a3611d9a9317ee7"
}
```

```bash
# Create the Security Group HTTP ingress rules.
aws ec2 authorize-security-group-ingress \
    --group-id sg-06a3611d9a9317ee7 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-007b20fdf5ec2f094",
            "GroupId": "sg-06a3611d9a9317ee7",
            "GroupOwnerId": "339712982725",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

```bash
# Create the Security Group SSH ingress rules.
aws ec2 authorize-security-group-ingress \
    --group-id sg-06a3611d9a9317ee7 \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-05160253f91c99605",
            "GroupId": "sg-06a3611d9a9317ee7",
            "GroupOwnerId": "339712982725",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

```bash
# Create User Data script
cat << EOF > /tmp/install-nginx.sh
#!/usr/bin/env bash
yum install -y nginx
systemctl enable --now nginx
EOF
```


```bash
# Launch EC2.
aws ec2 run-instances --image-id ami-0ebfd941bbafe70c6 \
  --count 1 \
  --instance-type t2.micro \
  --key-name devops-keypair \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-ec2}]' \
  --security-group-ids sg-06a3611d9a9317ee7 \
  --user-data file:///tmp/install-nginx.sh

```

<details>
  <summary><b>NOTE:</b> Click me for output.</summary>

```json
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0ebfd941bbafe70c6",
            "InstanceId": "i-04bea43952dd84312",
            "InstanceType": "t2.micro",
            "KeyName": "devops-keypair",
            "LaunchTime": "2024-09-30T05:39:57.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-81-126.ec2.internal",
            "PrivateIpAddress": "172.31.81.126",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-04dc9db3befca66a9",
            "VpcId": "vpc-062f238c2ca29e37a",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "3ff82a88-3f3b-44b8-981e-085aaa5f9e83",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2024-09-30T05:39:57.000Z",
                        "AttachmentId": "eni-attach-0a1b4a8e288952df3",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "devops-http-sg",
                            "GroupId": "sg-06a3611d9a9317ee7"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "12:c9:ca:45:03:cd",
                    "NetworkInterfaceId": "eni-0c977cd2d51cd3624",
                    "OwnerId": "339712982725",
                    "PrivateDnsName": "ip-172-31-81-126.ec2.internal",
                    "PrivateIpAddress": "172.31.81.126",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-81-126.ec2.internal",
                            "PrivateIpAddress": "172.31.81.126"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-04dc9db3befca66a9",
                    "VpcId": "vpc-062f238c2ca29e37a",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "devops-http-sg",
                    "GroupId": "sg-06a3611d9a9317ee7"
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
                    "Value": "devops-ec2"
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
    "OwnerId": "339712982725",
    "ReservationId": "r-0459fe0b55e473ae0"
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
            "AvailabilityZone": "us-east-1b",
            "InstanceId": "i-04bea43952dd84312",
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

```bash
# Get public information
aws ec2 describe-instances --instance-ids i-04bea43952dd84312 | grep -i public | tr -d ' ' | sort | uniq
```

```
"PublicDnsName":"ec2-52-207-132-79.compute-1.amazonaws.com",
"PublicIp":"52.207.132.79"
"PublicIpAddress":"52.207.132.79",
```

Check HTTP connection.

```bash
curl -Lv http://ec2-52-207-132-79.compute-1.amazonaws.com/
```

```html
*   Trying 52.207.132.79:80...
* Connected to ec2-52-207-132-79.compute-1.amazonaws.com (52.207.132.79) port 80 (#0)
> GET / HTTP/1.1
> Host: ec2-52-207-132-79.compute-1.amazonaws.com
> User-Agent: curl/7.74.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.24.0
< Date: Mon, 30 Sep 2024 05:42:07 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Thu, 05 Sep 2024 22:16:12 GMT
< Connection: keep-alive
< ETag: "66da2dac-267"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host ec2-52-207-132-79.compute-1.amazonaws.com left intact

```

We are done.