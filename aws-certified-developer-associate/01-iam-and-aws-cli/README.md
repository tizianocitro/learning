# 1 IAM & AWS CLI

**IAM (Identity and Access Management)** is a  global service. Therefore, there is no region to be selected, when you create a resource (user/group) in IAM, it will be available everywhere.

The **root account** (created by default,) should be used only for creating users. It should never be used or shared. It is identified by seeing only the account ID within the dashboard, while IAM Users have the username attached. Use the root account only to create your first IAM User and a few account/service management tasks. For everyday tasks, use an **IAM User**.

## 1.1 IAM Users & Groups

**IAM Users** are people within the organization, and can be grouped.

**IAM Groups** (also called IAM User Groups) only contain users, not other groups.

Users don’t have to belong to a group, and user can belong to multiple groups

![Examples of users and groups](/assets/aws-certified-developer-associate/users_groups.png "Examples of users and groups")

We create users to allow them to use our AWS account and to allow the to do so, we need to give them proper **permissions**.

When signing-in to AWS, you have to choose to login either with a IAM User or a Root account.

## 1.2 IAM Policies (Manage AWS Permissions)

Users or Groups can be assigned JSON documents called policies. **Policies** define the permissions of the user/group. 

In general, a policy is a JSON document that defines a set of permissions for making requests to AWS services, and can be used by IAM Users, IAM Groups, and IAM Roles.

In AWS you apply the **Least Privilege Principle**: don’t give more permissions than a user needs.

Following an example of policy:

```javascript
{
    "Id": "Policy-1",
    "Version": "2012-10-17",
    "Statement": [
        // Allow to use EC2 and call Describe
        {
            // Id of the statement, it is optional
            "Sid": "Statement-1",
            // Effect can assume two values: Allow or Deny
            "Effect": "Allow",
            // Describe* means any API that starts with Describe
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
                "cloudwatch:Describe*"
            ],
            "Resource": "*"
        },

        {
            "Effect": "Allow",
            "Principal": {
                "AWS": ["arn:aws:iam::<id>:<account>"]
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resources": ["arn:aws:s3::<bycket-name>/*"]
        }
    ]
}
```

## 1.3 Creating Users

IAM is a  global service. Therefore, when creating a user, it will be available in every region.

Upon creating a user, you can choose to give them access to the **AWS Management Console**.

It is also possible to choose between setting a password or autogenerating one if the user account will be used by someone else. At the same time, you can specify that the password must be changed at next sign-in.

You can add a user to a group (also creting the group during the user creation process). We can assign a policy to the group, so that the user will inherit the group permissions.

**Tags** can be added to the user. In general, they can be added to any resource in AWS.

User accounts can also be assigned with **aliases**. Aliases need to be unique and can be used instead of the account ID on sign-in.

## 1.4 IAM Policies

Policies attached to a group will apply to all users in the group. At the same time, deleting a group will remove the permissions to the users.

A policy attached to a single user is called **inline policy**.

![Policies for groups and users](/assets/aws-certified-developer-associate/policies.png "Policies for groups and users")

If a user belongs to multiple teams, they will inherit all the policies from all the groups.

AWS provides a lot of preconfigured permissions policies, such as *AdministratorAccess*, *IAMReadOnlyAccess* and *IAMFullAccess*.

Following is how the *AdministratorAccess* policy is defined by AWS
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

AWS provides a visual and a JSON editor for creating policies.

### 1.4.1 Policies Consist of
- **Id**: an identifier for the policy (optional).
- **Version**: policy language version, always include *"2012-10-17"*.
- **Statement**: one or more individual statements (required).

### 1.4.2 Statements Consist of
- **Sid**: the statement's identifier (optional).
- **Effect**: whether the statement allows or denies access (*Allow*, *Deny*)
- **Principal**: account/user/role to which this policy applied to.
- **Action**: list of actions this policy allows or denies.
- **Resource**: list of resources to which the actions applied to.
- **Condition**: conditions for when this policy is active (optional).

**Action** can contains APIs in the form of `<prefix>*`, meaning that all APIs that start with `<prefix>` will be included in the permission. For example, the following policy will allow any *List* API provided by IAM, such as ListUsers and ListGroups.
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:List*",
            "Resource": "*"
        }
    ]
}
```
Actions are expressed in the form `<service>:<API>`.

## 1.5 IAM Password Policy

In AWS, you can setup a password policy for:
- Minimum password length.
- Require specific character types:
    - Uppercase letters.
    - Lowercase letters.
    - Numbers.
    - Non-alphanumeric characters.
- Allow all IAM users to change their own passwords.
- Require users to change their password after some time (password expiration).
- Prevent password re-use.

The password policy can be set in the *Account Management -> Account Settings* menu.

## 1.6 Multi Factor Authentication (MFA)

Users can possibly change configurations or delete resources in your AWS account, so you want to protect your root account and IAM users using MFA. MFA can be activated by accessing the *Security Credentials* option in the user menu.

**MFA** means using a password alongside a device you own in a way that, even if the password is stolen or hacked, the account is not compromised as the malicious actor should also gain access to the device.

Options:
- **Virtual MFA Device**: Google Authenticator, Twilio Authy, etc. One application supports multiple token on a single phone.
- **Universal 2nd Factor (U2F) Security Key**: a phisical device like [Yubikey](https://it.wikipedia.org/wiki/YubiKey). One single security key is for multiple accounts and users.
- **Hardware Key Fob MFA Device**: another phisical device.
- **Hardware Key Fob MFA Device for AWS GovCloud (US)**: a phisical device for people working for the US government.

## 1.7 How to Access AWS

To access AWS, you have three options:
- AWS Management Console, which is protected by password and MFA.
- AWS Command Line Interface (CLI), which is protected by access keys.
- AWS Software Developer Kit (SDK), which is protected by access keys.

Access Keys are created through the AWS Console and it is responsibility of the users to manage their own access keys. To create an access key, you need to navigate to the *Security Credentials* under the *Users* menu.

You can access an access key only on creation, so it is important to safely store it. Upon creation, it is also possible to attach a description tag to the access key.

Each user will have:
- Access Key ID -> username
- Secret Access Key -> password

### 1.7.1 AWS CLI

The CLI enables you to interact with AWS services using commands in the shell, provides access to the public APIs of AWS services and can be used to develop scripts to manage resources on AWS.

It is open-source and available on [GitHub](https://github.com/aws/aws-cli).

Use the `aws configure` command to setup your CLI using a created access key.

### 1.7.2 AWS SDK

The SDK enables you to access and manage AWS services
programmatically within your application.

It supports many languages (JavaScript, Python, .NET, Java, Go, Node.js, ...), mobile SDKs (Android, iOS, ...), and IoT Device SDKs (Embedded C, Arduino, ...).

The AWS CLI itself is built on top of the AWS SDK for Python (Boto).

### 1.7.3 AWS CloudShell

It is a browser-based console available directly from within the AWS Console. Nice thing is that it will issue commands using the account you are signed in with and the region you have selected in the console.

You can download/upload files to the CloudShell environment as well as managing multiple tabs at the same time.

## 1.8 IAM Roles

Some AWS service will need to perform actions on your behalf, so they need permissions that we can assign to AWS services with **IAM Roles**. Some of the common roles are EC2 Instance Roles and Lambda Function Roles.

Roles are available under the *Account Management -> Roles* menu. Important for the exam are the roles for AWS services.

When creating a role for an AWS service, you need to choose to which service (or use case) you want to assign the role to. Then, you need to attach permissions policies to the role and give it a *name*.

Following an example for EC2:

![EC2 IAM Roles](/assets/aws-certified-developer-associate/ec2_iam_roles.png "EC2 IAM Roles")

After review and creation, the role is ready to be assigned to the target AWS services.

## 1.9 IAM Security Tools (Audit)

- **IAM Credentials Report (account-level)**: a report that lists all your AWS account's IAM users and the status of their credentials.
- **IAM Access Advisor (user-level)**: shows the service permissions granted to a user and when those services were last accessed. This information ca be used to revise policies because it may result that some users have access to more services than they need. For example, if a user never access EC2, it is a sign that they do not need it.

Use the dedicated entry *Credentials Report* in the *Access Reports* menu, which will provide a download for the report in CSV format.
Instead, *Access Advisor* is accessible from each user's dashboard.

## 1.10 Shared Responsibility Model for IAM

| AWS | You |
|-----|-----|
| Infrastructure | Users, Groups, Roles, Policies management and monitoring |
| Configuration and vulnerability analysis | Enable MFA on all accounts |
| Compliance validation | Rotate all your keys often |
| | Use IAM tools to apply appropriate permissions |
| | Analyze access patterns and review permissions |

## 1.11 IAM Best Practices

1. Don’t use the root account except for AWS account setup.
2. One physical user = One AWS user, so never share your AWS user.
3. Assign users to groups and assign permissions to groups (it is better than assigning permissions to users directly).
4. Create strong password policies.
5. Use and enforce the use of MFA, if possible.
6. Create and use IAM roles for giving permissions to AWS services.
7. Use Access Keys for programmatic access (CLI and SDK).
8. Regularly audit permissions of your account using IAM Credentials Report & IAM Access Advisor.
9. Never share IAM users and access keys.