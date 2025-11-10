# 21 CICD: CodeCommit, CodePipeline, CodeBuild, and CodeDeploy

Manual steps (e.g., create AWS resources manually) make it very likely to do mistakes. We would like our code in a repository and have it deployed onto AWS:
- Automatically.
- Correctly.
- Making sure it is tested before being deployed.
- With possibility to go into different stages (e.g., dev, test, staging, prod)
- With manual approval where needed.
