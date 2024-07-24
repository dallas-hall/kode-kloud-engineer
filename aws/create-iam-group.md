# AWS Create IAM Group

## Task

> The Nautilus DevOps team has been creating a couple of services on AWS cloud. They have been breaking down the migration into smaller tasks, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. Recently they came up with requirements mentioned below.
>
> Create an IAM group named `iamgroup_james`.

## Research

* AWS IAM Group
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html
  * https://docs.aws.amazon.com/cli/latest/reference/iam/create-group.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html or https://docs.aws.amazon.com/cli/latest/reference/iam/create-group.html

```bash
# Create the group.
aws iam create-group \
    --group-name iamgroup_james
```

```json
{
    "Group": {
        "Path": "/",
        "GroupName": "iamgroup_james",
        "GroupId": "AGPAYS2NVALYWKLXITJPG",
        "Arn": "arn:aws:iam::590183990001:group/iamgroup_james",
        "CreateDate": "2024-07-24T09:42:09Z"
    }
}
```
