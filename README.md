# Deploy Infrastructure using MCP CodeBuild

This repository contains CloudFormation templates to create an AWS CodeBuild project that uses Model Context Protocol (MCP) with AWS Bedrock to generate and deploy Terraform infrastructure from natural language instructions.

## Overview

The CloudFormation template (`codebuild-cloudformation.yaml`) creates:
- AWS CodeBuild project connected to GitHub with webhook triggers
- IAM role with necessary permissions for Bedrock, Terraform, and AWS services
- S3 bucket for build artifacts
- CloudWatch Log Group for build logs

The `buildspec.yml` file defines a build process that:
1. Installs Python, Terraform, and MCP dependencies
2. Uses AWS Bedrock to generate Terraform code from natural language instructions
3. Executes Terraform commands via AWS Terraform MCP Server
4. Deploys infrastructure automatically

## Prerequisites

1. AWS CLI configured with appropriate credentials
2. AWS Bedrock access with Anthropic Claude model enabled
3. GitHub repository (public or private)
4. IAM permissions to create CodeBuild projects, S3 buckets, and CloudWatch logs

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

## Build Specification

The `buildspec.yml` file defines the build process with the following phases:

### Install Phase
- Installs Python 3.11 runtime
- Upgrades pip
- Installs MCP Python SDK and AWS Terraform MCP Server
- Downloads and installs Terraform 1.6.6

### Pre-Build Phase
- Validates required environment variables
- Creates Terraform working directory
- Displays build configuration

### Build Phase
- Generates Terraform code using AWS Bedrock from natural language instructions
- Executes Terraform commands (init, validate, plan, apply) via MCP server
- Deploys infrastructure automatically

### Post-Build Phase
- Displays deployment completion message
- Lists generated Terraform files

### Artifacts
- Saves generated `main.tf` file
- Saves the Python script used for generation

## Environment Variables

The buildspec uses the following environment variables (configure in CodeBuild project or buildspec.yml):

- **AWS_REGION**: AWS region for deployment (default: `us-east-1`)
- **INSTRUCTION**: Natural language instruction for infrastructure creation (e.g., "Create an S3 bucket named my-company-logs-12345 with versioning enabled and block all public access.")
- **BEDROCK_MODEL_ID**: AWS Bedrock model ID (default: `anthropic.claude-3-5-sonnet-20241022-v2:0`)
- **TF_DIR**: Directory for Terraform files (default: `tfgen`)

## Running a Build

### Automatic Triggers

The CodeBuild project is configured with webhook triggers that automatically start builds on:
- Push events to the specified branch

### Manual Build

You can also trigger a build manually:

#### Using AWS Console

1. Navigate to CodeBuild service
2. Select your project
3. Click "Start build"

#### Using AWS CLI

```bash
aws codebuild start-build --project-name <project-name>
```

The project name follows the pattern: `<stack-name>-codebuild-project`

## Customization

### Modify Build Environment

Edit the CloudFormation template parameters:
- **BuildImage**: Change the Docker image used for builds
- **ComputeType**: Adjust compute resources
- **EnvironmentType**: Change environment type (Linux, Windows, ARM, etc.)

### Update Build Instructions

Modify the `INSTRUCTION` environment variable in `buildspec.yml` or override it in the CodeBuild project settings to change what infrastructure gets created.

### Change Bedrock Model

Update the `BEDROCK_MODEL_ID` environment variable to use a different AWS Bedrock model.

## Cleanup

To delete all resources:

```bash
aws cloudformation delete-stack --stack-name codebuild-github-project
```

**Note**: This will also delete any infrastructure created by Terraform during builds. Ensure you've backed up any important data.

## Troubleshooting

### Build Fails with Authentication Error

- Ensure the GitHub repository URL is correct
- Verify webhook permissions if using private repositories

### Permission Denied Errors

- Check that the IAM role has necessary permissions for:
  - Bedrock (invoke model)
  - Terraform operations (EC2, S3, IAM, etc.)
  - CloudWatch logs
  - S3 bucket access for artifacts

### Terraform Installation Fails

- Verify the CodeBuild image has `wget` and `unzip` available
- Check network connectivity to download Terraform

### Bedrock Model Access

- Ensure your AWS account has access to the specified Bedrock model
- Verify the model ID is correct for your region
- Check IAM permissions for Bedrock runtime operations

### Build Timeout

- Increase `TimeoutInMinutes` in the CloudFormation template
- Optimize your build commands in `buildspec.yml`
- Consider using a larger compute type for faster execution

## Security Notes

- The S3 bucket is configured with public access blocked
- IAM role includes necessary permissions for MCP and Terraform operations
- CloudWatch logs are retained for 7 days (configurable)
- Artifacts are stored in a versioned S3 bucket
- Terraform state is managed by the MCP server

## How It Works

1. **CodeBuild Trigger**: Webhook triggers a build on push to the repository
2. **Environment Setup**: CodeBuild installs Python, Terraform, and MCP dependencies
3. **Terraform Generation**: AWS Bedrock generates Terraform code from the natural language instruction
4. **MCP Execution**: The AWS Terraform MCP Server executes Terraform commands (init, validate, plan, apply)
5. **Infrastructure Deployment**: Terraform deploys the generated infrastructure
6. **Artifacts**: Generated Terraform files are saved to S3

## License

This project is open source and available for use.
