# AWS Delete IAM Group

## Task

> The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their AWS account. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their AWS environment.
>
> * Delete an IAM group named `iamgroup_mark`.

## Research

* AWS IAM Group
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage_delete.html
  * https://docs.aws.amazon.com/cli/latest/reference/iam/delete-group.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage_delete.html or https://docs.aws.amazon.com/cli/latest/reference/iam/delete-group.html

```bash
# Delete the group.
aws iam delete-group \
    --group-name iamgroup_mark
```
