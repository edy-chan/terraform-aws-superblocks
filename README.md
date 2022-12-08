<p align="center">
  <img src="./assets/logo.png" height="60"/>
</p>

<h1 align="center">Superblocks Terraform Module - AWS</h1>

<br/>

This document contains configuration and deployment details for deploying the Superblocks agent to AWS.

## Deploy with Terraform

### Install Terraform

To install Terraform on MacOS
```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Terraform officially supports `MacOS|Windows|Linux|FreeBSD|OpenBSD|Solaris`
Check out this https://developer.hashicorp.com/terraform/downloads for more details

### Deploy Superblocks On-Premise-Agent

#### Create your Terraform file

To get started, you'll need a `superblocks_agent_key`. To generate an agent key, go to the [Superblocks On-Premise Agent Setup Wizard](https://app.superblocks.com/opas)
```
module "terraform_aws_superblocks" {
  source  = "superblocksteam/terraform-aws-superblocks"
  version = ">=1.0"

  vpc_id         = "<VPC_ID>"
  lb_subnet_ids  = "<LIST_OF_SUBNET_IDS_FOR_LOAD_BALANCER>"
  ecs_subnet_ids = "<LIST_OF_SUBNET_IDS_FOR_SUPERBLOCKS_AGENT_ECS_CLUSTER>"
  domain         = "<DOMAIN>"
  subdomain      = "<SUBDOMAIN_FOR_SUPERBLOCKS_AGENT>"

  superblocks_agent_key = <SUPERBLOCKS_AGENT_KEY>""
}
```
To find your VPC use `aws ec2 describe-vpcs` or by finding them in the AWS management console.

#### Deploy
```
terraform init
terraform apply
```

### Advanced Configuration

#### Public Networking
ECS instances running the Superblocks On-Premise Agent are configured to receive traffic from a **private** Elastic Load Balancer. Allow public traffic to your On-Premise Agent by adding
```
lb_internal = false
```

#### VPC
The module by default deploys the OPA within an existing VPC. If you want your agent to access data across multiple VPCs, you can update the module to **create a new VPC** then set up VPC peering between the newly configured VPC and existing AWS VPCs. To update the module to create a new VPC
```
create_vpc = true
```

For more details on configuring VPC peering see [Connect VPCs using VPC peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html).

#### Security Group
A default security group will be created with an ingress cidr blocks `0.0.0.0/0`. To use your own security group
```
create_sg = false
security_group_ids = "<LIST_OF_YOUR_SECURITY_GROUP_IDS>"
```

#### Load Balancer
A new Elastic Load Balancer will be created by default to handle TLS termination before sending traffic to your ECS instances. To use an existing Loading Balancer, update the module to include
```
create_lb = false
lb_target_group_arn = "<YOUR_TARGET_GROUP_ARN>"
```
To find your target ground ARN use `aws elbv2 describe-target-groups` or by finding the Load Balancer in the AWS management console.

#### DNS & Certificate
If you use Route53 for domain management you can use the Terraform module to generate a DNS record and  certificate for your agent, and associated both with your Load Balancer. If you don't use Route53 or want to use an existing certificate & DNS record, add the following to your configuration
```
create_dns = false
certificate_arn = "<YOUR_CERTIFICATE_ARN>"
```
To find the certificate's ARN use `aws acm list-certificates` or by finding the Certificate in the AWS management console. For additional instructions on creating a certificate manually see [Issue and manage certificates](https://docs.aws.amazon.com/acm/latest/userguide/gs.html).

#### Instance Sized
Configure the CPU & memory limits allocated to your ECS instances use
```
container_cpu = 1024
container_memory = 4096
```

#### Scaling
AWS will automatically scale your ECS instances based on traffic. To configure the minimum and maximum number of instances the agent can scale to, set
```
container_min_capacity = 1
container_max_capacity = 10
```

#### Other Configurable Options
```
variable "superblocks_agent_environment" {
  type        = string
  default     = "*"
  description = <<EOF
    Use this varible to differentiate Superblocks Agent running environment.
    Valid values are "*", "staging" and "production"
  EOF
}

variable "superblocks_agent_image" {
  type        = string
  default     = "ghcr.io/superblocksteam/agent"
  description = "The docker image used by Superblocks Agent container instance"
}

variable "name_prefix" {
  type        = string
  default     = "superblocks"
  description = "This will be prepended to the name of each AWS resource created by this module"
}

variable "tags" {
  type        = map(string)
  default     = {}
  description = "A series of tags that will be added to each AWS resource created by this module"
}
```

