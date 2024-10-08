# AWS AutoMLOps Serverless Deployment

[![🧩 Serverless deployment to AWS](https://github.com/kwame-mintah/aws-automlops-serverless-deployment/actions/workflows/serverless-deploy.yml/badge.svg)](https://github.com/kwame-mintah/aws-automlops-serverless-deployment/actions/workflows/serverless-deploy.yml)
[![💥 Serverless remove resources](https://github.com/kwame-mintah/aws-automlops-serverless-deployment/actions/workflows/serverless-remove.yml/badge.svg)](https://github.com/kwame-mintah/aws-automlops-serverless-deployment/actions/workflows/serverless-remove.yml)

This repository contains the serverless deployment yaml for lambda functions used to achieve [MLOps Level 2 within AWS](https://aws.amazon.com/what-is/mlops/#seo-faq-pairs#how-to-implement-mlops-in-the-organization). Not all resources
are created by this deployment in AWS, some resources are created via Terraform in this repository [terraform-aws-machine-learning-pipeline](https://github.com/kwame-mintah/terraform-aws-machine-learning-pipeline) repository.

The lambda functions deployed aim to automate, preparing data and transforming features, training and tuning, deploying models and running inferences etc.

## Notice

This repository is to serve as an example, the architecture proposed may not apply to all use cases due to the limitations of lambdas. Please consider other AWS services,
for instance Elastic Container Service (ECS), for much better performance and longer running tasks. As the MLOps workflow has been split into various components,
it should be easy to identify areas that could benefit to being moved to a different service with better compute.

# Architecture

![proposed-automlops-level-2](/docs/drawio/aws-automlops-deployment.png)

1. User has received new data.
2. Data is uploaded to a GitHub repository.
3. A GitHub action is triggered uploading the data to a S3 Bucket.
4. Lambda function for data preprocessing is run due to an event trigger on the bucket.
5. Upon completion the transformed data is uploaded to another bucket.
6. Lambda function will trigger the SageMaker training job with various hyperparameters.
7. Training job is started using data split for training and validation.
8. Completed model is uploaded to a S3 Bucket.
9. Lambda function to deploy the new model for inference using serverless endpoint.
10. Message is sent to queue containing endpoint name and test data location.
11. Lambda function will invoke serverless endpoint with test data.
12. Results of predictions stored in a S3 bucket for Data scientist to examine.

## Lambda repositories

The source code for all lambda functions are stored in GitHub:

- dataPreProcessing: [aws-lambda-data-preprocessing](https://github.com/kwame-mintah/aws-lambda-data-preprocessing)
- modelTraining: [aws-lambda-model-training](https://github.com/kwame-mintah/aws-lambda-model-training)
- modelDeployment: [aws-lambda-model-deployment](https://github.com/kwame-mintah/aws-lambda-model-deployment)
- modelEvaluation: [aws-lambda-model-evaluation](https://github.com/kwame-mintah/aws-lambda-model-evaluation)

## GitHub Action (CI/CD)

The GitHub Action will deploy the lambda functions using the [serverless](https://github.com/serverless/github-action) action. Docker images used for deployment are stored
within an AWS ECR repository.

## Deploy serverless via CLI

If you wish to apply serverless changes via your machine instead of using GitHub Actions, please ensure the relevant AWS
credentials have been set as environment variables, terraform changes applied deploy, lambda versions environment
variables set and serverless CLI is installed.

> [!NOTE]
>
> Please see each lambdas GitHub tags to get the lambda version e.g. https://github.com/kwame-mintah/aws-lambda-data-preprocessing/tags, Would be `DATA_PREPROCESSING_VERSION=X.Y.Z`.

1. Install the [`serverless-iam-roles-per-function`](https://www.serverless.com/plugins/serverless-iam-roles-per-function) plugin:

   ```shell
   serverless plugin install --name serverless-iam-roles-per-function
   ```

2. Deploy resources to your chosen environment e.g. `dev`, `staging` or `prod`:

   ```shell
   serverless deploy --stage <env>
   ```

3. Remove deployed resources in an environment e.g. `dev`, `staging` or `prod`:

   ```shell
   serverless remove --stage <env>
   ```
