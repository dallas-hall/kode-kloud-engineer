# AWS Create Read-Only IAM Policy For EC2 Console Access

## Task

> When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements.
>
> Create an IAM policy named `iampolicy_mariyam` in `us-east-1` region, it must allow read-only access to the EC2 console, i.e this policy must allow users to view all instances, AMIs, and snapshots in the Amazon EC2 console.

## Research

* AWS IAM Policy
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-cli.html
  * https://docs.aws.amazon.com/cli/latest/reference/iam/create-policy.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html or https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-cli.html

```bash
# Create the policy file.
cat > policy.json
```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyActions",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:Get*",
        "ec2:ListImagesInRecycleBin",
        "ec2:ListSnapshotsInRecycleBin",
        "ec2:SearchLocalGatewayRoutes",
        "ec2:SearchTransitGatewayRoutes",
        "ec2messages:Get*"
      ],
      "Resource": "*"
    }
  ]
}
```

Close the file with control + d i.e. `^D`

```bash
# Create the policy.
aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document file://policy.json
```

```json
{
    "Policy": {
        "PolicyName": "iampolicy_mariyam",
        "PolicyId": "ANPAW3MEBSFAJFJAEAGN6",
        "Arn": "arn:aws:iam::471112716608:policy/iampolicy_mariyam",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-07-24T09:54:38Z",
        "UpdateDate": "2024-07-24T09:54:38Z"
    }
}
```
