# AWS Create Key Pair

## Task

> The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the AWS cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.
>
> For this task, create a key pair with the following requirements:
> * Name of the key pair should be `nautilus-kp`.
> * Key pair type must be `rsa`

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* AWS create key pair.
  * https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html

## Steps

Follow the steps at https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html

```bash
# Create the key pair
aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --key-format pem \
    --query "KeyMaterial" \
    --output text > nautilus-kp.pem
```
