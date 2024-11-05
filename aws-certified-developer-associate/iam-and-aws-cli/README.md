# IAM & AWS CLI

![Identity and Access Management](/assets/aws-certified-developer-associate/iam.png "Identity and Access Management"){ width="150" height="auto" style="display: block; margin: 0 auto" }

**IAM (Identity and Access Management)** is a  global service. Therefore, there is no region to be selected, when you create a resource (user/group) in IAM, it will be available everywhere.

The **root account** (created by default,) should be used only for creating users. It should never be used or shared. It is identified by seeing only the account ID within the dashboard, while IAM Users have the username attached.

## IAM Users & Groups

**Users** are people within the organization, and can be grouped.

**Groups** only contain users, not other groups.

Users don’t have to belong to a group, and user can belong to multiple groups

![Examples of users and groups](/assets/aws-certified-developer-associate/users_groups.png "Examples of users and groups"){ width="100%" height="auto" style="display: block; margin: 0 auto" }

We create users to allow them to use our AWS account and to allow the to do so, we need to give them proper **permissions**.

When signing-in to AWS, you have to choose to login either with a IAM User or a Root account.

## IAM Permissions

Users or Groups can be assigned JSON documents called policies. **Policies** define the permissions of the users. In AWS you apply the **Least Privilege Principle**: don’t give more permissions than a user needs.

Following an example of policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        // Allow to use EC2 and call Describe
        {
            "Effect": "Allow",
            "Action": "ec2:Describe*",
            "Resource": "*"
        },

        // Allow to use ELB and call Describe
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:Describe*",
            "Resource": "*",
        },

        // Allow to use CloudWatch and call ListMetrics,
        // GetMetricStatistics, and Describe
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:ListMetrics",
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:Describe*",
            ],
            "Resource": "*"
        }
    ]
}
```

## Creating Users

IAM is a  global service. Therefore, when creating a user, it will be available in every region.

Upon creating a user, you can choose to give them access to the **AWS Management Console**.

It is also possible to choose between setting a password or autogenerating one if the user account will be used by someone else. At the same time, you can specify that the password must be changed at next sign-in.

You can add a user to a group (also creting the group during the user creation process). We can assign a policy to the group, so that the user will inherit the group permissions.

**Tags** can be added to the user. In general, they can be added to any resource in AWS.

User accounts can also be assigned with **aliases**. Aliases need to be unique and can be used instead of the account ID on sign-in.