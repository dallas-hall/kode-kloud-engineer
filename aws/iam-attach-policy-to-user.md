# AWS Attach IAM Policy To User

## Task

> The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
>
> * An IAM user named `iamuser_kareem` and a policy named `iampolicy_kareem` already exist. Attach the IAM policy `iampolicy_kareem` to the IAM user `iamuser_kareem`.

## Research

* AWS add IAM policy
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policy-cli
  * https://docs.aws.amazon.com/cli/latest/reference/iam/list-policies.html
  * https://stackoverflow.com/questions/66287626/aws-cli-list-policies-to-find-a-policy-with-a-specific-name
  * https://docs.aws.amazon.com/cli/latest/reference/iam/attach-user-policy.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console or https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policy-cli

```bash
# Create the policy ID. Notice the backticks ` as the search term quotation.
aws iam list-policies \
  --query 'Policies[?PolicyName==`iampolicy_kareem`]'
```

```json
[
    {
        "PolicyName": "iampolicy_kareem",
        "PolicyId": "ANPA2UC3FO3OCY7UPEDUH",
        "Arn": "arn:aws:iam::730335639260:policy/iampolicy_kareem",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-07-25T07:09:37Z",
        "UpdateDate": "2024-07-25T07:09:37Z"
    }
]
```

```bash
# Attach policy to user.
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::730335639260:policy/iampolicy_kareem \
  --user-name iamuser_kareem
```
