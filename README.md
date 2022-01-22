# wordpress-multisite-ecs-efs-rds

Solution that deploys WordPress containers in AWS Elastic Container Service (ECS). These containers connect to Elastic File System (EFS) to persist and share the WordPress files between them and to Relational Database Service (RDS) to store the data.

## Architecture

![wordpress-multisite-ecs-efs-rds](https://user-images.githubusercontent.com/4935587/150462554-d7126f41-4155-4fa2-8041-f5c26297e26a.png)

## How to deploy solution

### Network requirements

1. VPC with an attached Internet gateway.
2. Two sets of one public subnet and one private subnet. Each set must belong to different availability zones.
    - The public subnet must route internet traffic through the VPC Internet gateway.
    - The public subnet must have a NAT gateway attached.
    - The private subnet must route internet traffic through the NAT gateway attached in the public subnet.

<details>
    <summary>Sample network stack</summary>

    AWSTemplateFormatVersion: '2010-09-09'
    Description: ''

    #################### STACK MAPPINGS ####################

    Mappings:

        SubnetConfig:
            VPC:
                CIDR: 10.2.0.0/16
            PublicSubnet1:
                CIDR: 10.2.0.0/24
            PublicSubnet2:
                CIDR: 10.2.1.0/24
            PrivateSubnet1:
                CIDR: 10.2.2.0/24
            PrivateSubnet2:
                CIDR: 10.2.3.0/24

    #################### STACK RESOURCES ####################

    Resources:

        #################### VPC ####################

        VPC:
            Type: AWS::EC2::VPC
            Properties:
                CidrBlock: !FindInMap [ SubnetConfig, VPC, CIDR ]
                InstanceTenancy: default
                EnableDnsHostnames: true
                EnableDnsSupport: true
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC'

        #################### INTERNET GATEWAY ####################

        InternetGateway:
            Type: AWS::EC2::InternetGateway
            Properties:
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-IG'
            
        InternetGatewayAttachment:
            Type: AWS::EC2::VPCGatewayAttachment
            Properties:
                InternetGatewayId: !Ref InternetGateway
                VpcId: !Ref VPC

        #################### PUBLIC ROUTE TABLE ####################

        PublicRouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
                VpcId: !Ref VPC
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubRT'

        DefaultPublicRoute:
            DependsOn:
                - InternetGatewayAttachment
            Type: AWS::EC2::Route
            Properties:
                RouteTableId: !Ref PublicRouteTable
                DestinationCidrBlock: 0.0.0.0/0
                GatewayId: !Ref InternetGateway

        #################### PUBLIC SUBNETS ####################

        #################### SUBNET1 ####################

        PublicSubnet1:
            Type: AWS::EC2::Subnet
            Properties:
                AvailabilityZone: !Select [ 0, !GetAZs '' ]
                CidrBlock: !FindInMap [ SubnetConfig, PublicSubnet1, CIDR ]
                MapPublicIpOnLaunch: true
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN1'
                VpcId: !Ref VPC

        PublicSubnet1RouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
                RouteTableId: !Ref PublicRouteTable
                SubnetId: !Ref PublicSubnet1

        PublicSubnet1ElasticIP:
            Type: AWS::EC2::EIP
            Properties:
                Domain: vpc
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN1-NG-EIP'
            
        PublicSubnet1NatGateway:
            Type: AWS::EC2::NatGateway
            Properties:
                AllocationId: !GetAtt PublicSubnet1ElasticIP.AllocationId
                SubnetId: !Ref PublicSubnet1
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN1-NG'

        #################### SUBNET2 ####################

        PublicSubnet2:
            Type: AWS::EC2::Subnet
            Properties:
                AvailabilityZone: !Select [ 1, !GetAZs '' ]
                CidrBlock: !FindInMap [ SubnetConfig, PublicSubnet2, CIDR ]
                MapPublicIpOnLaunch: true
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN2'
                VpcId: !Ref VPC

        PublicSubnet2RouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
                RouteTableId: !Ref PublicRouteTable
                SubnetId: !Ref PublicSubnet2

        PublicSubnet2ElasticIP:
            Type: AWS::EC2::EIP
            Properties:
                Domain: vpc
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN2-NG-EIP'
            
        PublicSubnet2NatGateway:
            Type: AWS::EC2::NatGateway
            Properties:
                AllocationId: !GetAtt PublicSubnet2ElasticIP.AllocationId
                SubnetId: !Ref PublicSubnet2
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PubSN2-NG'

        #################### PRIVATE SUBNETS ####################

        #################### SUBNET1 ####################

        PrivateSubnet1:
            Type: AWS::EC2::Subnet
            Properties:
                AvailabilityZone: !Select [ 0, !GetAZs '' ]
                CidrBlock: !FindInMap [ SubnetConfig, PrivateSubnet1, CIDR ]
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PrivSN1'
                VpcId:
                    Ref: VPC

        PrivateSubnet1RouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
                VpcId: !Ref VPC
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PrivSN1-RT'

        PrivateSubnet1RouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
                RouteTableId: !Ref PrivateSubnet1RouteTable
                SubnetId: !Ref PrivateSubnet1

        RouteToPublicSubnet1NatGateway:
            Type: AWS::EC2::Route
            Properties:
                RouteTableId: !Ref PrivateSubnet1RouteTable
                DestinationCidrBlock: 0.0.0.0/0
                NatGatewayId: !Ref PublicSubnet1NatGateway

        #################### SUBNET2 ####################

        PrivateSubnet2:
            Type: AWS::EC2::Subnet
            Properties:
                AvailabilityZone: !Select [ 1, !GetAZs '' ]
                CidrBlock: !FindInMap [ SubnetConfig, PrivateSubnet2, CIDR ]
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PrivSN2'
                VpcId:
                    Ref: VPC

        PrivateSubnet2RouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
                VpcId: !Ref VPC
                Tags:
                    - Key: Name
                    Value: !Sub '${AWS::StackName}-VPC-PrivSN2-RT'

        PrivateSubnet2RouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
                RouteTableId: !Ref PrivateSubnet2RouteTable
                SubnetId: !Ref PrivateSubnet2

        RouteToPublicSubnet2NatGateway:
            Type: AWS::EC2::Route
            Properties:
                RouteTableId: !Ref PrivateSubnet2RouteTable
                DestinationCidrBlock: 0.0.0.0/0
                NatGatewayId: !Ref PublicSubnet2NatGateway
                
</details>

### AWS Console

To deploy the solution using AWS Console, follow to this [tutorial](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

### AWS CLI

To deploy the solution using AWS CLI, follow these steps:
1. Clone this repo
2. Create a file called stack.env. Below you can find a sample of the contents of this file.
    ```
    VPCId=vpc-0aaa000aa00000a00
    PrivateSubnetIds=subnet-11bbbbbbb11bb111b,subnet-222c2cc22c2cccc2c
    PublicSubnetIds=subnet-333dd3333333d3d3d,subnet-444e444444d44d44d
    DatabaseInstanceClass=db.t3.micro
    DatabaseAllocatedStorage=20
    DatabaseMaxAllocatedStorage=40
    DatabaseBackupRetentionPeriod=5
    EnableDatabaseMultiAZ=Yes
    EnableDatabaseReadReplica=Yes
    DatabaseSecretRotationSchedule=30
    EnableEFSAutomaticBackups=Yes
    EFSPerformanceMode=generalPurpose
    EFSThroughputMode=bursting
    EFSProvisionedThroughputInMibps=
    EnableCustomDomain=Yes
    CustomDomain=wp.example.com
    CustomDomainCertificateARN=arn:aws:acm:us-east-1:111111111111:certificate/0c2189d1-b4ab-4dce-aebb-2a90b0332acd
    ECSTaskvCPU=.5
    ECSTaskMemory=3072
    ECSLogRetentionPeriod=1
    ECSServiceAutoScalingMetric=AverageCPUUtilization
    ECSServiceAutoScalingTargetValue=80
    ECSServiceAutoScalingTargetMinCapacity=1
    ECSServiceAutoScalingTargetMaxCapacity=2
    ```
    :warning: Optional parameters can be left empty.

3. Deploy the stack
    ```
    aws cloudformation deploy --template-file ./template.yaml --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND --parameter-overrides $(cat stack.env) --stack-name <NAME YOUR STACK>
    ```

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