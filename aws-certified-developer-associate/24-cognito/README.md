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
