# Launch EC2 Instance & Copy SSH Key

## Task

> The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host (`aws-client`). The instance should be of type `t2.micro` and named `devops-ec2`. A new SSH key should be created on the `aws-client` host if it doesn't already exist. This key should then be added to the authorized keys of the `root` user on the EC2 instance, allowing password-less SSH access from the aws-client host.
>
## Research

* AWS EC2
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
  * https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html
  * https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html
* SSH
  * https://serverfault.com/questions/955036/ssh-copy-id-doesnt-work-with-pem-files
  * https://serverfault.com/questions/706336/how-to-get-a-pem-file-from-ssh-key-pair

## Steps

Follow the steps at https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name devops-ec2 \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > ~/.ssh/devops-ec2

# Change permissions
chmod 600 ~/.ssh/devops-ec2

# Get the public key
ssh-keygen -f ~/.ssh/devops-ec2 -y > ~/.ssh/devops-ec2.pub

# Launch EC2.
aws ec2 run-instances --image-id ami-0cd59ecaf368e5ccf \
  --count 1 \
  --instance-type t2.micro \
  --key-name devops-ec2 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-ec2}]' > ec2.json

# Get the instance ID
grep -i instanceid ec2.json
```

```
            "InstanceId": "i-0ac665a655c9c0f09",
```

```bash
# Check EC2 status.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1c",
            "InstanceId": "i-05c2a1aaa33954275",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "InstanceStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "initializing"
                    }
                ],
                "Status": "initializing"
            },
            "SystemStatus": {
                "Details": [
                    {
                        "Name": "reachability",
                        "Status": "initializing"
                    }
                ],
                "Status": "initializing"
            }
        }
    ]
}
```

Wait a few minutes for it to start up.

```bash
# Check EC2 status.
aws ec2 describe-instance-status
```

```json
{
    "InstanceStatuses": [
        {
            "AvailabilityZone": "us-east-1c",
            "InstanceId": "i-0ac665a655c9c0f09",
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
# Get the public domain.
aws ec2 describe-instances --instance-ids i-0ac665a655c9c0f09 | grep -i publicdns
```

```
                    "PublicDnsName": "ec2-3-84-141-97.compute-1.amazonaws.com",
                                "PublicDnsName": "ec2-3-84-141-97.compute-1.amazonaws.com",
                                        "PublicDnsName": "ec2-3-84-141-97.compute-1.amazonaws.com",
```


```bash
# Add the key to SSH agent.
ssh-add ~/.ssh/devops-ec2

# Copy the SSH key to the EC2 instance for root.
ssh-copy-id -i ~/.ssh/devops-ec2 root@ec2-3-84-141-97.compute-1.amazonaws.com
```

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/devops-ec2.pub"
The authenticity of host 'ec2-3-84-141-97.compute-1.amazonaws.com (3.84.141.97)' can't be established.
ECDSA key fingerprint is SHA256:zm29JXm9QsXv4JOL98Te2YlZyHG62T/uHl3nkCgXmdQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Please login as the user "ubuntu" rather than the user "root".
```

The return message said to use `ubuntu` instead.

```bash
# Copy the SSH key to the EC2 instance for ubuntu.
ssh-copy-id -i ~/.ssh/devops-ec2 ubunut@ec2-3-84-141-97.compute-1.amazonaws.com

# Connect
ssh ubunut@ec2-3-84-141-97.compute-1.amazonaws.com -t 'sudo cat /root/.ssh/authorized_keys'

# Check the authorized_keys file for root
sudo cat /root/.ssh/authorized_keys
```

```
no-port-forwarding,no-agent-forwarding,no-X11-forwarding,command="echo 'Please login as the user \"ubuntu\" rather than the user \"root\".';echo;sleep 10;exit 142" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCAdje9NnkOFoqeKKzEG/3bhs+bz49kaQbxjOyX1NAwZnFZGIeLdfZaodfoVN9zE/njst5GMg+68smLF4dZrp1rJM3c88bb/7WITcUc5vp9n7R46nqrPK8pV85/ft9K8nQOxf1WW9ihOLlGrcNdKGNuPXfBDokazMVFTwvNObm79OkbpcwO/BWcwiqdxd0DuB8VU1RLLDxFOgEC5GSkaCJu4TlPaJwueL17qmbnUYz+alSMqMgoekrIMID2MSd3alKX8fAaxNxX9cN663tL8mlnLoT5aCIs8Rvl8qAKEGNqyXJUPdMwRLocqftkiO7zmmi/V5kR57B4Om6UdOZGY2iT devops-ec2
```

We are done.