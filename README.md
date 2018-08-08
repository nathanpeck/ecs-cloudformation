# Deploy containers using Elastic Container Service and CloudFormation

ECS and Fargate give you a lot of control over how you want to deploy containers. There are three common patterns, which many customers use:

1) A service with direct access to the internet, and exposed publically to the internet so that people can access it. The service has a public IP address it can initiate direct communication to other external services:

   ![public subnet public lb](images/public-subnet-public-lb.png)

2) A service protected inside a private subnet, with no direct internet access. Because the service does not have public IP address it must initiate outbound connections through a NAT Gateway, which communicates to the external internet on the service's behalf. However, you may still want to give the public limited access to the service via a load balancer which is public:

   ![private subnet public lb](images/private-subnet-public-lb.png)

3) A service which is protected inside a private subnet. Not only does it not have a private IP address, but it also has a private load balancer which can only be accessed by your own services. This is often used for internal services, where one frontend service communicates to a backend service which the public is not intended to directly access. In the diagram below notice how someone from the public internet initiates the blue connection to the public facing service in the public subnet, but that service can then initiate a green connection the private internal service:

   ![private subnet private lb](images/private-subnet-private-lb.png)

This repository contains templates for deploying all three architectures across both ECS on EC2 and ECS on Fargate.

&nbsp;

&nbsp;

## Step 1: Create a service linked role for ECS

Use the AWS CLI to execute the following command. This will create a role that enables ECS on your account.

```
aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
```

&nbsp;

&nbsp;

## Step 2: Deploy a cluster

There are four cluster variations to choose between depending on your needs:

__EC2 Hosted:__

EC2 hosted clusters give you the most control over the type of EC2 instance that hosts your containers, and allows you to take advantage of reserved instances:

* [EC2 Cluster in fully public network](cluster/cluster-ec2-public-vpc.yml) (This cluster hosts all containers in a public subnet with direct internet access, but protected by security group rules that provide a firewall the instances.)
* [EC2 Cluster with private network](cluster/cluster-ec2-private-vpc.yml) (This cluster hosts the containers in a private subnet so they have no direct internet access. Instead all inbound traffic must travel in through an ingress, and all outgoing traffic must go out through an NAT gateway.)

__Fargate Hosted:__

Fargate clusters allow you to run containers on-demand without hosting any EC2 instances on your account. AWS manages the underlying infrastructure types and capacity, operating system and software patches.

* [Fargate cluster with fully public network](cluster/cluster-fargate-public-vpc.yml) (This cluster only runs containers in a public subnet with public IP addresses, although the containers can be protected by a security group firewall.)
* [Fargate cluster with private network](cluster/cluster-fargate-private-vpc.yml) (This cluster can run containers in a private subnet without a public IP address, but outbound traffic goes through an NAT gateway.)

&nbsp;

&nbsp;

## Step 3: Add one or more ingresses

In order for traffic to reach your containers you need an ingress. You can deploy a public external ingress that anyone on the internet can use to reach your containers, or a private internal ingress that can only be used by your own containers inside your cluster to reach other containers inside the cluster.

__Load Balancers:__

* [External, public ALB](ingress/alb-external.yml) (An external ALB is ideal for public services like a website or an API that you want other clients on the internet to access. This load balancer can be added to both public and privately networked clusters.)
* [Internal, private ALB](ingress/alb-internal.yml) (An internal ALB only accepts traffic from other containers in your cluster, and is ideal for protecting internal backend API's. This load balancer requires a privately networked Fargate or EC2 cluster. If you try to deploy this type of load balancer in a "public" type cluster it will fail because the cluster does not have any private subnets to host it.)

&nbsp;

&nbsp;

## Step 4: Add one or more services

Now that you have a cluster and one or more ingresses you can deploy services:

__Public/Private EC2 cluster + Public Load Balancer:__

Deploy the [public EC2 service template](service/service-ec2-public-lb.yml) to host containers in the cluster, with a public facing load balancer in front of them.

__Public/Private EC2 cluster + Private Load Balancer:__

Deploy the [private EC2 service template](service/service-ec2-private-lb.yml) host containers cluster behind a private internal load balancer that only accepts traffic from other containers in the cluster.

__Public Fargate cluster + Public Load Balancer:__

Deploy the [Fargate public subnet, public LB service template](service/service-fargate-public-subnet-public-lb.yml) to host containers in a public subnet with direct internet access and a public facing load balancer in front of them.

__Private Fargate cluster + Public Load Balancer:__

Deploy the [Fargate private subnet, public LB service template](service/service-fargate-private-subnet-public-lb.yml) to host containers in a private subnet with no direct internet access, but with a public facing load balancer that can still get public traffic to them.

__Private Fargate cluster + Private Load Balancer:__

Deploy the [Fargate private subnet, private LB service template](service/service-fargate-private-subnet-private-lb.yml) to host containers in a private subnet with no direct internet access, and put them behind a private, internal load balancer which only accepts traffic from other internal containers.

&nbsp;

&nbsp;

## Step 5: Check the ingress stack outputs for your service URL

Once the service stack is deployed check the outputs tab of the ingress stack to get the URL to use to access your containers. Note that an external ALB's URL will be accessible to the public from any computer, but an internal ALB's URL will only be accessible if you make the request from an instance inside the VPC, with the appropriate security group. If you want to test spin up a Cloud 9 development environment in the VPC, add its security group to the load balancer's security group and execute a `curl` command.

&nbsp;

&nbsp;

## Further customizations

Note that these baseline templates have only HTTP listeners (no SSL support) but this can be easily added to the templates once you create or import an SSL certificate into Amazon Certificate Manager. Additionally, you may want to customize the default autoscaling rules that are embedded in the service template.
