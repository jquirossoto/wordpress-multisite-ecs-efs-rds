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