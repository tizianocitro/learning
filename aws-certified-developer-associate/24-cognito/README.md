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

### 24.2.1 Accessing Users and Groups

You can **access users** by clicking on *Users* in the side-bar from the user pool details:

![CUP Users](/assets/aws-certified-developer-associate/cup_users.png)

From there, you can also **create users** by clicking on *Create User*:

![CUP Create Users](/assets/aws-certified-developer-associate/cup_create_users.png)

You can **access groups** where you can group users by clicking on *Groups* in the side-bar from the user pool details:

![CUP Groups](/assets/aws-certified-developer-associate/cup_groups.png)

You can also **access sign-in and sign-up** to configure options like MFA, user verification (e.g., SMS), self-service sign-up, and more.

### 24.2.2 Accessing Authentication Domains

You can **access the authentication domain** by clicking on *Domain* in the side-bar from the user pool details:

![CUP Domain](/assets/aws-certified-developer-associate/cup_domain.png)

You can also configure custom domains with Route 53.

### 24.2.3 Configuring Federated Identities

You can **configure federated identities** by clicking on *Social and External Providers* in the side-bar from the user pool details:

![CUP Federated Identities](/assets/aws-certified-developer-associate/cup_federated_identities.png)

Then, click on *Add Identity Provider* to **configure a provider** of your choice (e.g., Amazon, Apple, Google):

![CUP Configure Federated Identities](/assets/aws-certified-developer-associate/cup_configure_federated_identities.png)

### 24.2.4 Configuring Authentication Methods

You can **configure authentication methods** by clicking on *Authentication Methods* in the side-bar from the user pool details:

![CUP Authentication Methods](/assets/aws-certified-developer-associate/cup_authentication_methods.png)

For example, to use emails:

![CUP Set Email Auth](/assets/aws-certified-developer-associate/cup_set_email_auth.png)

You can also **set password policies and passkeys**:

![CUP Password Policies and Passkeys](/assets/aws-certified-developer-associate/cup_password_policies_and_passkeys.png)

### 24.2.5 Using Extensions to Trigger Lambda Functions

Access the extensions by clicking on *Extensions* in the side-bar from the user pool details.

From this page, you can **configure Lambda functions to be triggered by certain events** like sign-up or authentication.

![CUP Extensions](/assets/aws-certified-developer-associate/cup_extensions.png)

### 24.2.6 Accessing Other Settings

You can access the **application clients** by clicking on *App Clients* in the side-bar from the user pool details:

![CUP App Clients](/assets/aws-certified-developer-associate/cup_app_clients.png)

For **security settings**, you can enable WAF, threat protection, and log streaming.

For the **branding settings**, you can configure the template for the messages that Cognito sends to users or the style of the sign-in/sign-up page.
