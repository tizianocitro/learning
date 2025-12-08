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

## 21.5 CodePipeline

It is a visual workflow tool to orchestrate CICD pipelines.
- Sources: CodeCommit, ECR, S3, Bitbucket, GitHub, etc.
- Build phase: CodeBuild, Jenkins, CloudBees, TeamCity.
- Test phase: CodeBuild, AWS Device Farm, 3rd party tools, etc.
- Deploy phase: CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3, etc.
- Invoke phase: Lambda, Step Functions.

You can build stages, where each stage can have sequential actions and/or parallel actions.
- For example, a stage can involve `Build -> Test -> Deploy -> Load Testing -> ...`.
- Manual approval can be defined at any stage.

### 21.5.1 CodePipeline Artifacts

Each pipeline stage can create **artifacts**, which are stored in an S3 bucket and passed on to the next stage.

![CodePipeline Artifacts](/assets/aws-certified-developer-associate/cicd_codepipeline_artifacts.png "CodePipeline Artifacts")

### 21.5.2 Troubleshooting CodePipeline

For CodePipeline pipeline/action/stage execution state changes, you can use Amazon EventBridge. For example, you can create events for failed pipelines or cancelled stages.
- If CodePipeline fails a stage, the pipeline stops, and you can get information in the console.
- If CodePipeline cannot perform an action, make sure the IAM role attached has enough IAM permissions (IAM policy).

Additionally, CloudTrail can be used to audit AWS API calls.

## 21.6 Using CodePipeline to Deploy Changes to Elastic Beanstalk

Create an Elastic Beanstalk application and two environments `dev` and `prod`.

Go to CodePipeline and create a new pipelineby clicking on the *Create Pipeline* button:

![CodePipeline Create](/assets/aws-certified-developer-associate/cicd_codepipeline_create.png "CodePipeline Create")

Select the **creation process**:

![CodePipeline Creation Process](/assets/aws-certified-developer-associate/cicd_codepipeline_create.png "CodePipeline Creation Process")

Assign it a **name** and select the **execution mode**:

![CodePipeline Name](/assets/aws-certified-developer-associate/cicd_codepipeline_name.png "CodePipeline Name")

And define the **service role**:

![CodePipeline Service Role](/assets/aws-certified-developer-associate/cicd_codepipeline_service_role.png "CodePipeline Service Role")

You can eventually setup **variables** for the pipeline:

![CodePipeline Variables](/assets/aws-certified-developer-associate/cicd_codepipeline_variables.png "CodePipeline Variables")

Next, select the **source provider**. In this case, we will use GitHub (Version 2):

![CodePipeline Source Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_source_provider.png "CodePipeline Source Provider")

For this, you need to create a GitHub connection by clicking on the *Connect to GitHub* button:

![CodePipeline GitHub Connection](/assets/aws-certified-developer-associate/cicd_codepipeline_github_connection.png "CodePipeline GitHub Connection")

And enter it, alongside the **repository name** and **branch name**:

![CodePipeline GitHub Connection Name](/assets/aws-certified-developer-associate/cicd_codepipeline_github_connection_name.png "CodePipeline GitHub Connection Name")

Configure triggers for the pipeline. You can select to trigger the pipeline on every push (or pull request) to the repository, or only on specific branches or files:

![CodePipeline GitHub Triggers](/assets/aws-certified-developer-associate/cicd_codepipeline_github_triggers.png "CodePipeline GitHub Triggers")

![CodePipeline GitHub Triggers 2](/assets/aws-certified-developer-associate/cicd_codepipeline_github_triggers_2.png "CodePipeline GitHub Triggers 2")

You can then select the **build provider**, but we will skip this step for now (we will see more later):

![CodePipeline Build Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_build_provider.png "CodePipeline Build Provider")

Next is the **deploy provider**:

![CodePipeline Deploy Provider](/assets/aws-certified-developer-associate/cicd_codepipeline_deploy_provider.png "CodePipeline Deploy Provider")

In this case, we will use Elastic Beanstalk to deploy to the `dev` environment, which requires the following information:

![CodePipeline Deploy Provider Beanstalk](/assets/aws-certified-developer-associate/cicd_codepipeline_deploy_provider_eb.png "CodePipeline Deploy Provider Beanstalk")

You can finally **create the pipeline**, and **it will be created and started**:

![CodePipeline Created](/assets/aws-certified-developer-associate/cicd_codepipeline_created.png "CodePipeline Created")

Until all stages are completed:

![CodePipeline Created 2](/assets/aws-certified-developer-associate/cicd_codepipeline_created_2.png "CodePipeline Created 2")

### 21.6.1 Adding Stages to CodePipeline

You can add stages to the pipeline by clicking on the *Edit* button and then clicking on the *Add Stage* button of the phase you want to add the new stage to:

![CodePipeline Add Stage](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage.png "CodePipeline Add Stage")

Give the new stage a **name**:

![CodePipeline Add Stage Name](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_name.png "CodePipeline Add Stage Name")

Configure the **action group** for the stage:

![CodePipeline Add Stage Action Group](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_name.png "CodePipeline Add Stage Action Group")

We will select Beanstalk as the action provider:

![CodePipeline Add Stage Action Group Beanstalk](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_action_group_eb.png "CodePipeline Add Stage Action Group Beanstalk")

And enter the specific information it requires:

![CodePipeline Add Stage Action Group Beanstalk Info](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_action_group_eb_info.png "CodePipeline Add Stage Action Group Beanstalk Info")

**Multiple action groups can be added to a stage**. This is useful for when you need a **manual approval** step like in this case because we are deploying to production:

![CodePipeline Add Stage Manual Approval](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_manual_approval.png "CodePipeline Add Stage Manual Approval")

So, it will appear that you need to approve the deployment before it is done:

![CodePipeline Add Stage Manual Approval 2](/assets/aws-certified-developer-associate/cicd_codepipeline_add_stage_manual_approval_2.png "CodePipeline Add Stage Manual Approval 2")

Save the pipeline and it will be updated:

![CodePipeline Updated](/assets/aws-certified-developer-associate/cicd_codepipeline_updated.png "CodePipeline Updated")

Now, push a change to the GitHub repository and it will trigger the pipeline. This time, it will deployu to the `dev` environment and then **wait for a manual approval** to deploy to the `prod` environment:

![CodePipeline Waiting for Approval](/assets/aws-certified-developer-associate/cicd_codepipeline_waiting_for_approval.png "CodePipeline Waiting for Approval")

So, review the changes in the `dev` environment and click on the *Review* button to approve the deployment:

![CodePipeline Review Changes](/assets/aws-certified-developer-associate/cicd_codepipeline_review_changes.png "CodePipeline Review Changes")

And submit the approval so that the deployment to the `prod` environment can continue.
