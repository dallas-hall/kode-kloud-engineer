# Launch EC2 Instance & Copy SSH Key

## Task

> The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host (`aws-client`). The instance should be of type `t2.micro` and named `xfusion-ec2`. A new SSH key should be created on the `aws-client` host if it doesn't already exist. This key should then be added to the authorized keys of the `root` user on the EC2 instance, allowing password-less SSH access from the aws-client host.
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
    --key-name xfusion-ec2 \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > ~/.ssh/xfusion-ec2

# Change permissions
chmod 600 ~/.ssh/xfusion-ec2

# Get the public key
ssh-keygen -f ~/.ssh/xfusion-ec2 -y > ~/.ssh/xfusion-ec2.pub

# Launch EC2.
aws ec2 run-instances --image-id ami-0cd59ecaf368e5ccf \
  --count 1 \
  --instance-type t2.micro \
  --key-name xfusion-ec2 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' > ec2.json

# Get the instance ID
grep -i instanceid ec2.json | tr -d ' '
```

```
"InstanceId":"i-0c536e1443816d1ce",
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
            "InstanceId": "i-0c536e1443816d1ce",
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
            "AvailabilityZone": "us-east-1b",
            "InstanceId": "i-0c536e1443816d1ce",
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
aws ec2 describe-instances --instance-ids i-0c536e1443816d1ce | grep -i publicdns | tr -d ' ' | uniq
```

```
"PublicDnsName":"ec2-54-210-47-132.compute-1.amazonaws.com",
```


```bash
# Add the key to SSH agent.
ssh-add ~/.ssh/xfusion-ec2
```

```
Identity added: /root/.ssh/xfusion-ec2 (/root/.ssh/xfusion-ec2)
```

```bash
# Copy the SSH key to the EC2 instance for root.
ssh-copy-id -i ~/.ssh/xfusion-ec2 root@ec2-54-210-47-132.compute-1.amazonaws.com
```

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/xfusion-ec2.pub"
The authenticity of host 'ec2-54-210-47-132.compute-1.amazonaws.com (54.210.47.132)' can't be established.
ECDSA key fingerprint is SHA256:EAD6Kyy/vIsgqEeRHjsVctjHCp+zMbz177FssNmWVvg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Please login as the user "ubuntu" rather than the user "root".
```

The return message said to use `ubuntu` instead.

```bash
# Copy the SSH key to the EC2 instance for ubuntu.
ssh-copy-id -i ~/.ssh/xfusion-ec2 ubuntu@ec2-54-210-47-132.compute-1.amazonaws.com
```

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/xfusion-ec2.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed

/usr/bin/ssh-copy-id: WARNING: All keys were skipped because they already exist on the remote system.
                (if you think this is a mistake, you may want to use -f option)
```

I was able to log in after this but the task still failed because `root` cannot log in like in the message above. The SSH keys already existed and it seems like that is from when I created the EC2 instance with them, so I am going to log in and manually add the key and override the existing keys.

```bash
# Get the key
cat ~/.ssh/xfusion-ec2.pub
```

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCax/Db6euIq+MJoBK72wbleb9UuNPcPUdCDdjAhMORe6nitr0OfaVWO2j1ATlxqkd02+s2UKp9ZhRBx1M1ldDn0V3TJn+xTLifEXFOZGn0cY9iPkpXUApj6cYc2DIHnNku5bNiqid0aD2BjnOjxxjouFTC+9tv01t/ijgWiNOMLHL3H5uXxcWbvzENpl8lzg4yAXbz+okreoyjIst55bs1/iNmsGryfHFE92uGJrXlgvVfWHzrLWwh5HFomY11eKNQSXL1z10te5H/r+0j9mYd3sYbnyuy/y6RmxkcAkbGTskL2cl2GFXlKwOhREQMWOoisJQJiLJySQ/Yt3NUXI8j
```

```bash
# Login and switch to root
ssh ubuntu@ec2-54-210-47-132.compute-1.amazonaws.com
sudo -i

# Update the SSH key file for root
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCax/Db6euIq+MJoBK72wbleb9UuNPcPUdCDdjAhMORe6nitr0OfaVWO2j1ATlxqkd02+s2UKp9ZhRBx1M1ldDn0V3TJn+xTLifEXFOZGn0cY9iPkpXUApj6cYc2DIHnNku5bNiqid0aD2BjnOjxxjouFTC+9tv01t/ijgWiNOMLHL3H5uXxcWbvzENpl8lzg4yAXbz+okreoyjIst55bs1/iNmsGryfHFE92uGJrXlgvVfWHzrLWwh5HFomY11eKNQSXL1z10te5H/r+0j9mYd3sYbnyuy/y6RmxkcAkbGTskL2cl2GFXlKwOhREQMWOoisJQJiLJySQ/Yt3NUXI8j' > ~/.ssh/authorized_keys

# Check the file
cat ~/.ssh/authorized_keys
```

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCax/Db6euIq+MJoBK72wbleb9UuNPcPUdCDdjAhMORe6nitr0OfaVWO2j1ATlxqkd02+s2UKp9ZhRBx1M1ldDn0V3TJn+xTLifEXFOZGn0cY9iPkpXUApj6cYc2DIHnNku5bNiqid0aD2BjnOjxxjouFTC+9tv01t/ijgWiNOMLHL3H5uXxcWbvzENpl8lzg4yAXbz+okreoyjIst55bs1/iNmsGryfHFE92uGJrXlgvVfWHzrLWwh5HFomY11eKNQSXL1z10te5H/r+0j9mYd3sYbnyuy/y6RmxkcAkbGTskL2cl2GFXlKwOhREQMWOoisJQJiLJySQ/Yt3NUXI8j
```

```bash
# Logout from root and the node
exit
exit

# SSH as root
ssh root@ec2-54-210-47-132.compute-1.amazonaws.com
```

```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1055-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Wed Sep  4 23:11:12 UTC 2024

  System load:  0.0               Processes:             97
  Usage of /:   21.5% of 7.57GB   Users logged in:       0
  Memory usage: 21%               IPv4 address for eth0: 172.31.80.252
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


root@ip-172-31-80-252:~#
```

We are done.
