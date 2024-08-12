# Allocate & Associate Elastic IP To EC2 Instance

## Task

> The Nautilus DevOps Team has received a new request from the Development Team to set up a new EC2 instance. This instance will be used to host a new application that requires a stable IP address. To ensure that the instance has a consistent public IP, an Elastic IP address needs to be associated with it. The instance will be named `nautilus-ec2`, and the Elastic IP will be named `nautilus-eip`. This setup will help the Development Team to have a reliable and consistent access point for their application.
>
> Create an EC2 instance named `nautilus-ec2` using any linux AMI like ubuntu, the Instance type must be `t2.micro` and associate an Elastic IP address with this instance, name it as `nautilus-eip`.

## Research

* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
* AWS Elastic IP
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-eips.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-address.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html and https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-eips.html or https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html and https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/working-with-eips.html

```bash
# Get an Amazon Linux AMI ID, the output is many pages long.
aws ec2 describe-images --owners amazon --filters "Name=architecture,Values=x86_64" > amazon-ami.json

# Pick a random one
less amazon-ami.json

# Create the key pair
aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > nautilus-kp.pem

# Launch EC2.
aws ec2 run-instances --image-id ami-0cd59ecaf368e5ccf \
  --count 1 \
  --instance-type t2.micro \
  --key-name nautilus-kp \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]'

```

```json
# Lots of output omitted.
"InstanceId": "i-071130686bee992f6"
```

```bash
# Create Elastic IP
aws ec2 allocate-address --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=nautilus-eip}]'
```

```json
{
  "PublicIp": "23.22.91.236",
  "AllocationId": "eipalloc-0e06fec3e39dcce90",
  "PublicIpv4Pool": "amazon",
  "NetworkBorderGroup": "us-east-1",
  "Domain": "vpc"
}
```

```bash
# Associate Elastic IP
aws ec2 associate-address --instance-id i-071130686bee992f6 \
  --allocation-id eipalloc-0e06fec3e39dcce90
```

```json
{
  "AssociationId": "eipassoc-0f7d53fc8ee939dd6"
}
```

