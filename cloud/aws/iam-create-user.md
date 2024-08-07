# AWS Create IAM User

## Task

> When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements:
>
> For this task, create an IAM user named `iamuser_kareem`.

## Research

* AWS IAM User
  * https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html

## Steps

Follow the steps at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html

```bash
# Create the user.
aws iam create-user \
    --user-name iamuser_kareem
```

```json
{
    "User": {
        "Path": "/",
        "UserName": "iamuser_kareem",
        "UserId": "AIDAYS2NWWCESKA7V55TY",
        "Arn": "arn:aws:iam::590184099977:user/iamuser_kareem",
        "CreateDate": "2024-07-24T09:36:46Z"
    }
}
```
