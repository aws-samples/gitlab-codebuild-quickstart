# GitLab-CodeBuild Quick Start

This CloudFormation stack helps you to integrate your GitLab to AWS CodeBuild quickly.

## Architecture

![Architecture](./img/arch.png)

## Usage

1. Click the following button to launch the CloudFormation Stack in your account.

   [![Launch Stack](./img/launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template?stackName=GitLab-CodeBuild&templateURL=https://wjinghui-public.s3.amazonaws.com/templates/gitlab-codebuild-quickstart.yaml)

2. (Optional) Modify your AWS CodeBuild service role to add permissions like Amazon ECR access. You can find the role ARN in the output of this stack if you use `admin` or `minimal` as the input parameter `RoleArnForCodeBuild`. You can always modify this role later.
3. Modify your repository to adapt your build command. The default build command is `bash build.sh`, in this case your need a file `build.sh` in your repository to specify your build steps.
4. Create a webhook in your GitLab project settings.
   1. For the webhook URL, use the value of the output `WebhookURL` of the CloudFormation stack.
   2. For the secret token, use the value of the parameter `WebhookSecretToken` of the CloudFormation stack.
   3. Add webhook triggers as you need.
5. When your webhook is triggered, you will find your builds in the AWS CodeBuild console.

## Example

Here is an example of `build.sh` if you want to build a docker image and upload it to Amazon ECR:

```bash
# install docker
apt update -y
apt install docker -y

# login to AWS ECR
aws ecr get-login-password --region us-east-1 | docker login \
    --username AWS --password-stdin \
    1234567890.dkr.ecr.us-east-1.amazonaws.com

# build image
docker build -t myImage .
docker tag myImage 1234567890.dkr.ecr.us-east-1.amazonaws.com/myImage

# create ECR registry
aws ecr create-repository --repository-name=myImage

# publish image
docker push 1234567890.dkr.ecr.us-east-1.amazonaws.com/myImage
```

## Security Concerns

- The parameters `GitLabPassword` and `WebhookSecretToken` are stored in the AWS Secrets Manager.
- **The AWS CodeBuild project has the permission to retrieve `GitLabPassword` from the AWS Secrets Manager to download your source codes.** The GitLab password will be stored as an environment variable and can be accessed by your build command.

## Limits

Since this is a quick start template, many AWS CodeBuild features are not included, but you can configure them by yourself.

Those AWS CodeBuild features includes:

- Access assets within your VPCs like RDS, ElastiCache, etc.
- Cache build assets to accelerate later builds.
- Generate build badge.
- Set timeout and queued timeout.
- Use certificates from your Amazon S3 bucket.
- Attach to Amazon EFS file systems.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

