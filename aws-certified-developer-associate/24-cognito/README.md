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

## 24.3 Other Features in Cognito User Pools

### 24.3.1 Lambda triggers

CUP can invoke a Lambda function synchronously on these triggers:

![CUP Lambda Triggers](/assets/aws-certified-developer-associate/cup_lambda_triggers.png)

### 24.3.2 Hosted Authentication UI

Cognito has a hosted authentication UI that can be added to applications to handle sign-up and sign-in.
- It provides a foundation for integration with social logins, OIDC or SAML.
- It can be customized with a custom logo and custom CSS.

You can have custom domains for the UI. To do so, you must create an ACM certificate in `us-east-1`.
- The custom domain must be defined in the *App Integration* section because it is a general configuration that applies to all application client.

### 24.3.3 Adaptive Authentication

This feature allows you to block sign-ins or require MFA if the login appears suspicious.

The way it works, Cognito examines each sign-in attempt and generates a risk score (`low`, `medium`, `high`) for how likely it is that the sign-in request comes from malicious attacker.
- Users are prompted for MFA only when risk is detected.
- Risk score is based on different factors: if the user has used the same device, location, or IP address.
- It also checks for compromised credentials and in case triggers the account takeover protections by sending phone and email notification.
- It integrates with CloudWatch Logs to log sign-in attempts, risk score, failed challenges, etc.

![CUP Adaptive Authentication](/assets/aws-certified-developer-associate/cup_adaptive_authentication.png)

### 24.3.4 JSON Web Token

CUP issues JWT tokens (Base64 encoded) that include:
- Header.
- Payload.
- Signature.

The signature must be verified to ensure that the JWT can be trusted.
- There are libraries that can help you verify the validity of JWT issued by CUP.

The payload will contain the user information, some of which are:
- `sub` UUID: it is the ID of the user in Cognito database, which allows you to retrieve even more information about the user.
- `email`.
- `cognito:username`.

## 24.4 Adding Authentication in ALB

The **ALB can securely authenticate users**, which benefits because your application can:
- Offload the work of authenticating users to the load balancer.
- Focus on the business logic.

ALB can authenticate users through:
- Identity Providers (IdPs): they need to be compliant with OpenID Connect (OIDC).
- Cognito User Pools that allows you to support:
    - Social IdPs like Amazon, Facebook, or Google.
    - Corporate identities using SAML, LDAP, or Microsoft Active Directory.

For this to work, you must configure an HTTPS listener to set `authenticate-oidc` and `authenticate-cognito` rules.

![CUP_ALB](/assets/aws-certified-developer-associate/cup_alb.png)

To **handle unauthenticated requests** we can configure `OnUnauthenticatedRequest` to:
- Ask to authenticate (default).
- Deny the request.
- Allow the request.

### 24.4.1 ALB with Cognito Authentication

To configure it:
- Create a user pool, client and domain.
- Make sure the ID token is returned as JWT.
- Add the social or Corporate IdP, if needed.
- Configure the URL redirections, which are necessary.
- Allow your user pool domain on your IdP application's callback URL. For example:
    - https://domain-prefix.auth.region.amazoncognito.com/saml2/idpresponse.
    - https://user-pool-domain/oauth2/idpresponse.

![ALB with Cognito Authentication](/assets/aws-certified-developer-associate/alb_with_cognito_authentication.png)

### 24.4.2 ALB with OIDC Authentication

This is a more complex authentication process, following the OAuth 2.0 standard.

![ALB with OIDC Authentication](/assets/aws-certified-developer-associate/alb_with_oidc_authentication.png)

To configure it:
1. Configure a client ID and secret.
2. Allow redirect from OIDC to your ALB DNS name
(AWS-provided) and CNAME (DNS Alias of your application):
    - https://DNS/oauth2/idpresponse.
    - https://CNAME/oauth2/idpresponse.
3. Set the parameters shown in the image below.

![ALB with OIDC Authentication Configuration](/assets/aws-certified-developer-associate/alb_with_oidc_authentication_configuration.png)
