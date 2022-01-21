# wordpress-multisite-ecs-efs-rds

Solution that deploys WordPress containers in AWS Elastic Container Service (ECS). These containers connect to Elastic File System (EFS) to persist and share the WordPress files between them and to Relational Database Service (RDS) to store the data.

## Architecture

![wordpress-multisite-ecs-efs-rds](https://user-images.githubusercontent.com/4935587/150461988-55f637fc-062d-4704-b191-add19ae837d5.png)

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