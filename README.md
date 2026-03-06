

# AWS-VPC ArchitectureVPC
![AWS_VPC.drawio.svg](images/AWS_VPC.drawio.svg)

## Features
- VPC with configurable CIDR block and DNS hostname support
- Public, private, and database subnet tiers across multiple Availability Zones
- Internet Gateway for public subnets
- NAT Gateway (with Elastic IP) for private and database subnet outbound access
- Separate route tables for each subnet tier
- Optional VPC peering support flag
- Fully customizable tags on every resource

## Steps

1. Terraform initializes the AWS provider – Terraform downloads the AWS provider plugin and connects to AWS using the configured region.
2. Input variables are loaded – Values like project name, environment, VPC CIDR, and subnet CIDRs are read and used to parameterize the infrastructure.
3. VPC is created – The main network (aws_vpc) is created using the provided CIDR block; all networking resources depend on this VPC.
4. Internet Gateway is attached to the VPC – The Internet Gateway enables internet connectivity for resources inside the VPC.
5. Availability Zones are fetched – Terraform retrieves available AZs in the region to distribute subnets for high availability.
6. Public subnets are created – Public subnets are created inside the VPC across multiple AZs and configured to assign public IPs to instances.
7. Private subnets are created – Private subnets are created in the same VPC but without direct internet exposure.
8. Elastic IP for NAT Gateway is allocated – A static public IP is reserved so the NAT Gateway can communicate with the internet.
9. NAT Gateway is created in a public subnet – The NAT Gateway uses the Elastic IP and allows private subnet resources to access the internet.
10. Public route table is created – This route table controls internet routing for public subnets.
11. Route to Internet Gateway is added – A route directing all outbound traffic `(0.0.0.0/0)` from public subnets to the Internet Gateway is configured.
12. Private route table is created – A separate route table is created to manage traffic from private subnets.
13. Route to NAT Gateway is added – Private subnets send internet-bound traffic to the NAT Gateway for outbound connectivity.
14. Public route table is associated with public subnets – This association enables instances in public subnets to access the internet through the Internet Gateway.
15. Private route table is associated with private subnets – This association allows instances in private subnets to access the internet securely via the NAT Gateway.

## Requirements

| Name      | Version |
|-----------|---------|
| terraform | >= 1.0  |
| aws       | >= 5.0  |


## Provider

| Name | Version |
|------|---------|
| aws  | >= 5.0  |


## Inputs

| Name                      | Description                                                                         | Type         | Default                            | Required |
|---------------------------|-------------------------------------------------------------------------------------|--------------|------------------------------------|----------|
| project                   | Project name used for naming and tagging resources                                  | string       | n/a                                | yes      |
| environment               | Deployment environment. Must be one of: dev, qa, uat, prod                          | string       | n/a                                | yes      |
| vpc_cidr                  | CIDR block for the VPC                                                              | string       | `"10.0.0.0/16"`                    | no       |
| public_subnet_cidrs       | List of CIDR blocks for public subnets. One subnet is created per AZ                | list(string) | `["10.0.1.0/24", "10.0.2.0/24"]`   | no       |
| private_subnet_cidrs      | List of CIDR blocks for private subnets. One subnet is created per AZ               | list(string) | `["10.0.11.0/24", "10.0.12.0/24"]` | no       |
| database_subnet_cidrs     | List of CIDR blocks for database subnets. One subnet is created per AZ              | list(string) | `["10.0.21.0/24", "10.0.22.0/24"]` | no       |
| vpc_tags                  | Additional tags to merge on the VPC resource                                        | map(string)  | `{}`                               | no       |
| igw_tags                  | Additional tags to merge on the Internet Gateway                                    | map(string)  | `{}`                               | no       |
| public_subnet_tags        | Additional tags to merge on public subnets                                          | map(string)  | `{}`                               | no       |
| private_subnet_tags       | Additional tags to merge on private subnets                                         | map(string)  | `{}`                               | no       |
| database_subnet_tags      | Additional tags to merge on database subnets                                        | map(string)  | `{}`                               | no       |
| public_route_table_tags   | Additional tags to merge on the public route table                                  | map(string)  | `{}`                               | no       |
| private_route_table_tags  | Additional tags to merge on the private route table                                 | map(string)  | `{}`                               | no       |
| database_route_table_tags | Additional tags to merge on the database route table                                | map(string)  | `{}`                               | no       |
| eip_tags                  | Additional tags to merge on the Elastic IP                                          | map(string)  | `{}`                               | no       |
| nat_gateway_tags          | Additional tags to merge on the NAT Gateway                                         | map(string)  | `{}`                               | no       |
| is_peering_required       | Whether VPC peering is required. Enables peering-related resources when set to true | bool         | `false`                            | no       |

## Outputs

> Outputs should be defined in your `outputs.tf`. Common outputs from this module include:

| Name                | Description                    |
|---------------------|--------------------------------|
| vpc_id              | The ID of the VPC              |
| public_subnet_ids   | List of public subnet IDs      |
| private_subnet_ids  | List of private subnet IDs     |
| database_subnet_ids | List of database subnet IDs    |
| nat_gateway_id      | The ID of the NAT Gateway      |
| internet_gateway_id | The ID of the Internet Gateway |