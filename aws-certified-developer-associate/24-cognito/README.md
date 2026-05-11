# 24 Cognito

Cognito is used to **provide users we do not know yet an identity to interact with our web or mobile applications**.

Two sub-services: Cognito User Pools and Cognito Identity Pools (used to be called Federated Identity).

**Cognito User Pools (CUP)**:
- Provides a sign in functionality for application users.
- Integrates with API Gateway and ALB.

**Cognito Identity Pools**:
- Provides AWS credentials to users so they can access AWS resources directly.
- Integrates with Cognito User Pools as an identity provider.

**Cognito vs IAM**: Cognito is for users that are outside of AWS. And is for cases like:
- Hundreds of users or even more.
- Mobile and web users.
- Need for authentication with SAML, OpenID Connect and other protocols.

## 24.1 Cognito User Pools

CPU allows you to **create a serverless database of users for your web and mobile applications**.

It offers:
- **Simple login**: login with username (or email) and password combination.
- **Login with federated identities**: login users from Facebook, Google, SAML, etc.
- Logins sends back a JSON Web Token (JWT).

With support for:
- Password reset.
- Email and phone number verification.
- Multi-factor authentication.
- Blocking users if their credentials are compromised elsewhere.

![CUP](/assets/aws-certified-developer-associate/cup.png)

### 24.1.1 Cognito User Pools Integrations

CUP integrates with API Gateway and ALB natively.

**With API Gateway**:

![CUP With API Gateway](/assets/aws-certified-developer-associate/cup_with_ag.png)

**With ALB**:

![CUP With ALB](/assets/aws-certified-developer-associate/cup_with_alb.png)

## 24.2 Creating User Pools in Cognito

Go to Cognito console and click on *User Pools* to access your user pools:

![CUP User Pools](/assets/aws-certified-developer-associate/cup_user_pools.png)

Then, click on *Create User Pool* to start creating a pool:

![CUP Creation](/assets/aws-certified-developer-associate/cup_creation.png)

So, the first step is to select the **application type** and give your application a **name**:

![CUP Name](/assets/aws-certified-developer-associate/cup_name.png)

Next, configure the **sign in options** (email, phone number, and username):

![CUP Sign In Options](/assets/aws-certified-developer-associate/cup_sign_in_options.png)

And the **return URL** that defines where to redirect users after they successfully sign in:

![CUP Return URL](/assets/aws-certified-developer-associate/cup_return_url.png)

With this, you can create your user pool:

![CUP Created](/assets/aws-certified-developer-associate/cup_created.png)

Click on it and you will **access the user pool details**:

![CUP Details](/assets/aws-certified-developer-associate/cup_details.png)
