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
