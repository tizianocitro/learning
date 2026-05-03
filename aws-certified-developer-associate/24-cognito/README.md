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
