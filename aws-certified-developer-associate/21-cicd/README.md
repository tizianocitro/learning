# 21 CICD: CodeCommit, CodePipeline, CodeBuild, and CodeDeploy

Manual steps (e.g., create AWS resources manually) make it very likely to do mistakes. We would like our code in a repository and have it deployed onto AWS:
- Automatically.
- Correctly.
- Making sure it is tested before being deployed.
- With possibility to go into different stages (e.g., dev, test, staging, prod)
- With manual approval where needed.

## 21.1 Technology Stack for CI/CD on AWS

For all of this, we need a **CI/CD pipeline**. AWS has several services to help us with this:
- `CodeCommit`: store code.
- `CodePipeline`: automate pipelines.
- `CodeBuild`: build and test code.
- `CodeDeploy`: deploy the code (no Elastic Beanstalk).
- `CodeStar`: manage software development activities in one place.
- `CodeArtifact`: store, publish, and share software packages.
- `CodeGuru`: automated code reviews using machine learning.

![Technology Stack](/assets/aws-certified-developer-associate/cicd_techstack.png "Technology Stack")

## 21.2 Continuous Integration (CI)

Developers push the code to a code repository often (e.g., GitHub, CodeCommit, Bitbucket). A testing/build server checks the code as soon as it is pushed (e.g., CodeBuild, Jenkins), builds and tests it. The developer gets feedback about the tests and checks that have passed/failed.

Advantages of CI:
- Find and fix bugs early.
- Deliver faster as the code is tested and deploy often.

![Continuous Integration](/assets/aws-certified-developer-associate/cicd_ci.png "Continuous Integration")

## 21.3 Continuous Delivery (CD)

Ensures that:
- Software can be released reliably whenever needed.
- Deployments happen often and are quick.

CD usually means automated deployment (e.g., CodeDeploy, Spinnaker).

![Continuous Delivery](/assets/aws-certified-developer-associate/cicd_cd.png "Continuous Delivery")

## 21.4 CodeCommit

Version control is the ability to understand the various changes that happened to the code over time, and possibly roll back. All these are enabled by using a version control system like Git.

CodeCommit is fully managed and highly available. It provides:
- Private Git repositories.
- No size limit on repositories (scale seamlessly). 
- Code availability only in AWS account, increasing security and compliance.
- Security by encryption, access control, etc.
- Integration with Jenkins, CodeBuild, and other CI tools.

### 21.4.1 CodeCommit Security

**Interactions** are done using Git.

**Authentication** via:
- SSH Keys: users can configure SSH keys in their IAM console.
- HTTPS: CLI credential helper or Git credentials for IAM users.

**Authorization** via IAM policies to manage user/role permissions to repositories.

**Encryption**:
- Repositories are automatically encrypted at rest using KMS.
- Encrypted in transit: you can only use HTTPS or SSH, which are both secure.

**Cross-account access**:
- Do not share SSH keys or your AWS credentials.
- Use an IAM role in your AWS account and use STS (`AssumeRole` API).

### 21.4.2 CodeCommit vs GitHub

![CodeCommit vs GitHub](/assets/aws-certified-developer-associate/cicd_codecommit_vs_github.png "CodeCommit vs GitHub")

### 21.4.3 CodeCommit is Deprecated

On July 25th 2024, AWS discontinued CodeCommit, so new customers cannot use the service and AWS recommends to migrate to an external Git solution.
- For example, use GitHub or GitLab instead.
