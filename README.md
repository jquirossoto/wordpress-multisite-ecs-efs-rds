# wordpress-ecs-efs-rds

Solution that deploys WordPress containers in AWS Elastic Container Service (ECS). These containers connect to Elastic File System (EFS) to persist and share the WordPress files between them and to Relational Database Service (RDS) to store the data.

:warning: This CloudFormation stack its not production ready! For example:

1. The database has been configured to reduce expenses.
3. The VPC configuration might not fit your requirements.
4. The AutoScaling policy might not fit your requirements.

## Architecture

![wordpress-ecs-efs-rds](https://user-images.githubusercontent.com/4935587/149641137-dcede996-8228-4f76-8f7c-59146b851e49.png)

## How to run the solution locally

### Things you will need

1. [Composer](https://getcomposer.org/download/)
2. [Docker](https://docs.docker.com/get-docker/)

To run the solution locally, execute the following procedure:

1. Clone this repo
2. Start the database container

```
composer run docker-up
```

3. Run the WordPress container

```
composer run docker-run
```

## How to deploy and run the solution in AWS

:warning: If you deploy this solution, AWS will start charging you for the following services:

1. NatGateway
2. RDS
3. Secrets Manager

### Things you will need

1. [AWS account](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start)
2. [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

To deploy and run the solution in AWS, execute the following procedure:

1. Clone this repo
2. Create create a wordpress.env file in the root of the project. The contents of this file will include the parameters for the CloudFormation template:

- **AutoScalingMinCapacity**: Specify the maximum value that you plan to scale out to. Default: 1
- **AutoScalingMaxCapacity**: Specify the minimum value that you plan to scale in to. Default: 2


The contents of the wordpress.env file should look like this:

```
AutoScalingMinCapacity=1
AutoScalingMaxCapacity=2
```

3. Verify that the VPC CIDR configuration defined in the CloudFormation template does not conflict with any other VPC in your AWS account.
4. Make sure the credentials that you will use to deploy the CloudFormation stack have enough permissions to create all of the resources.
5. Deploy the CloudFormation stack:

```
composer run cf-stack-deploy
```
6. :confetti_ball: Visit your WordPress URL. To obtain the URL, run the following command. The URL is in the Outputs section.

```
composer run cf-stack-describe
```

7. If you don't want the solution deployed anymore, delete the CloudFormation stack. Make sure the ECR repository and the open-api bucket are empty.

```
composer run cf:stack:delete
```