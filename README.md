# AWS Elastic Container Service (ECS) ARM templates

We host our stateless Laravel applications containers in ECS. 
We recently moved our applications from Intel-based images to ARM-based images.

We used these CloudFormation templates to build the ARM-based ECS cluster 
and each ECS service. Refer to repos from past presentations, such as 
[sgf-aws/aws-ecs-demo-laravel](https://github.com/sgf-aws/aws-ecs-demo-laravel)
for sample ``Dockerfile`` and ``buildspec.yml`` files.

We are publishing these CloudFormation templates because we could NOT find ANY 
CloudFormation templates for ECS on ARM (as of December 2020).
Most templates are based on "Amazon Linux 1", which does NOT offer an ARM image.
These templates are derived from various existing CloudFormation templates
published by AWS as well as original research required to launch an ECS cluster 
on ARM with "Amazon Linux 2".

## Limitations

The ONLY known limitation with this template is the inability to update outdated
ECS agents in the EC2 instances from the ECS console. It is unclear if the ECS
console is unable to interact with the newer Amazon Linux 2 OS image or if this
is an ARM-specific issue.

My workaround was to add a "LaunchTemplateCounter" value to the template that, 
when incremented, forces CloudFormation to deploy new EC2 instances with the 
latest available "Amazon Linux 2" image and latest ECS agent. This can be a slow
process, since CF will replace (start new, stop old) a single EC2 instance at a time.

## Create the ECS Cluster

The following CloudFormation template will create an ECS cluster and ARM-based
EC2 hosts for the cluster.
You can override various options, including the default EC2 instance type 
(t4g.small) and the number of EC2 instances that will host this ECS cluster.

```
cloudformation/ecs-cluster.yml
```
NOTE: You must specify the following existing resources. This template does NOT create these resources:
* Existing VPC ID
* Existing Subnet IDs
* Optional Existing EC2 SSH Key Pair

## Create an ECS Web Service

The following CloudFormation template will create a web service designed to run
a Laravel web application. If you have multiple Laravel-based web services (e.g.
microservices), you will use this template to create a separate CloudFormation
stack for each web service.

```
cloudformation/ecs-service-web.yml
```

NOTE: You must specify the following existing resources. This template does NOT create these resources:
* Existing VPC ID
* Existing ECS Cluster ID (see first template above)
* Existing Task Container Image Path (e.g. ECR Path)
* Existing ALB Listener Path
* Existing Primary Hostname (hostname must already be pointed to ALB)
* Optional Existing Secondary Hostname (hostname must already be pointed to ALB)

## Create an ECS Worker Service

The following CloudFormation template will create a worker service designed to
run either a Laravel Cron worker, a Laravel Queue worker, or a 
[Laravel Plain Queue](https://github.com/dusterio/laravel-plain-sqs) worker.

```
cloudformation/ecs-service-worker.yml
```

NOTE: You must specify the following existing resources. This template does NOT create these resources:
* Existing VPC ID
* Existing ECS Cluster ID (see first template above)
* Existing Task Container Image Path (e.g. ECR Path)
* Existing SQS Queue Names

## Cleanup

You must delete the following items BEFORE you can delete your CloudFormation
Stacks:

* None?

When you delete your CloudFormation stacks, most items will be automatically 
deleted. The following items must be deleted manually:

* CloudWatch Log Groups
* Any items you created manually must be deleted manually.
