# AWS Delete IAM Role

## Task

> The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their AWS account. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their AWS environment.
>
> * Delete the IAM role named `iamrole_james`.

## Research

* AWS IAM Role
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html

```bash
# Delete the role.
aws iam delete-role \
    --role-name iamrole_james
```
