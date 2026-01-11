# Deploy Infrastructure using MCP CodeBuild

This repository contains CloudFormation templates to create an AWS CodeBuild project connected to a GitHub repository.

## Overview

The CloudFormation template (`codebuild-cloudformation.yaml`) creates:
- AWS CodeBuild project connected to GitHub
- IAM role with necessary permissions
- S3 bucket for build artifacts
- CloudWatch Log Group for build logs

## Prerequisites

1. AWS CLI configured with appropriate credentials
2. GitHub repository access (public or private)
3. For private repositories: GitHub personal access token or AWS CodeStar Connections setup

## Deployment Steps

### Option 1: Using AWS Console

1. Log in to the AWS Management Console
2. Navigate to CloudFormation service
3. Click "Create stack" → "With new resources (standard)"
4. Upload the `codebuild-cloudformation.yaml` file
5. Fill in the parameters:
   - **GitHubRepoUrl**: `https://github.com/Sathyafish/deploy-infra-using-mcp-codebuild`
   - **GitHubBranch**: `main` (or your preferred branch)
   - **BuildImage**: Choose appropriate build image (default: `aws/codebuild/standard:7.0`)
   - **ComputeType**: Choose compute type (default: `BUILD_GENERAL1_SMALL`)
6. Review and create the stack

### Option 2: Using AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name codebuild-github-project \
  --template-body file://codebuild-cloudformation.yaml \
  --parameters ParameterKey=GitHubRepoUrl,ParameterValue=https://github.com/Sathyafish/deploy-infra-using-mcp-codebuild \
               ParameterKey=GitHubBranch,ParameterValue=main \
  --capabilities CAPABILITY_NAMED_IAM
```

### Option 3: Using AWS CLI with parameters file

Create a `parameters.json` file:

```json
[
  {
    "ParameterKey": "GitHubRepoUrl",
    "ParameterValue": "https://github.com/Sathyafish/deploy-infra-using-mcp-codebuild"
  },
  {
    "ParameterKey": "GitHubBranch",
    "ParameterValue": "main"
  },
  {
    "ParameterKey": "BuildImage",
    "ParameterValue": "aws/codebuild/standard:7.0"
  },
  {
    "ParameterKey": "ComputeType",
    "ParameterValue": "BUILD_GENERAL1_SMALL"
  }
]
```

Then deploy:

```bash
aws cloudformation create-stack \
  --stack-name codebuild-github-project \
  --template-body file://codebuild-cloudformation.yaml \
  --parameters file://parameters.json \
  --capabilities CAPABILITY_NAMED_IAM
```

## GitHub Connection Setup

### For Public Repositories

The template uses OAuth authentication which works automatically for public repositories. No additional setup is required.

### For Private Repositories

You have two options:

#### Option A: Using AWS CodeStar Connections (Recommended)

1. Go to AWS CodeStar Connections in the AWS Console
2. Create a new connection to GitHub
3. Authorize the connection in GitHub
4. Update the CloudFormation template to use the connection ARN instead of OAuth

#### Option B: Using Personal Access Token

1. Generate a GitHub personal access token with `repo` scope
2. Store it in AWS Secrets Manager or Systems Manager Parameter Store
3. Update the CloudFormation template to reference the secret

## Build Specification

The project includes a sample `buildspec.yml` file that defines the build process. Customize it according to your project needs:

- **pre_build**: Commands to run before the build
- **build**: Main build commands
- **post_build**: Commands to run after the build
- **artifacts**: Files to be saved as build artifacts

## Running a Build

After deployment, you can trigger a build:

### Using AWS Console

1. Navigate to CodeBuild service
2. Select your project
3. Click "Start build"

### Using AWS CLI

```bash
aws codebuild start-build --project-name <project-name>
```

The project name can be found in the CloudFormation stack outputs.

## Stack Outputs

After successful deployment, the stack provides:

- **CodeBuildProjectName**: Name of the CodeBuild project
- **CodeBuildProjectArn**: ARN of the CodeBuild project
- **CodeBuildServiceRoleArn**: ARN of the IAM role
- **CodeBuildArtifactsBucket**: S3 bucket name for artifacts
- **CodeBuildBadgeUrl**: URL for the build status badge

## Customization

### Modify Build Environment

Edit the CloudFormation template parameters:
- **BuildImage**: Change the Docker image used for builds
- **ComputeType**: Adjust compute resources
- **EnvironmentType**: Change environment type (Linux, Windows, ARM, etc.)

### Add Environment Variables

Add environment variables in the `EnvironmentVariables` section of the CodeBuild project:

```yaml
EnvironmentVariables:
  - Name: MY_VAR
    Value: my-value
  - Name: SECRET_VAR
    Value: !Ref SecretParameter
    Type: PARAMETER_STORE
```

## Cleanup

To delete all resources:

```bash
aws cloudformation delete-stack --stack-name codebuild-github-project
```

## Troubleshooting

### Build Fails with Authentication Error

- For public repos: Ensure the repository URL is correct
- For private repos: Set up CodeStar Connections or use a personal access token

### Permission Denied Errors

- Check that the IAM role has necessary permissions
- Verify S3 bucket permissions for artifacts

### Build Timeout

- Increase `TimeoutInMinutes` in the CloudFormation template
- Optimize your build commands in `buildspec.yml`

## Security Notes

- The S3 bucket is configured with public access blocked
- IAM role follows least privilege principle
- CloudWatch logs are retained for 7 days (configurable)
- Artifacts are stored in a versioned S3 bucket

## License

This project is open source and available for use.
