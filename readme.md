# aws_manage_route_tables

## Description:

Manages AWS Route Tables as follows:
- Running the [main.yml](./tasks/main.yml) play will create the Route Tables defined in `aws_route_tables`.  
- Running the [get_route_table_facts.yml](./tasks/get_route_table_facts.yml) play will return information about the specified Route Table.

## Dependencies

This role uses the AWS cli.  It therefore requires AWS credentials as environment variables or in a configuration file (see [here](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#cli-aws-help-config-vars)).  The role `aws_credentials` will create a config file for use with the AWS cli.

## Behaviour:

**Feature:** Create AWS Route Tables
- **Given** valid AWS credentials
- **Given** an existing VPC
- **Given** existing subnets (if required)
- **When** the script is executed
- **Then** a Route Table is created
- **Then** routes are added to the Route Table
- **Then** subnets are assigned to the Route Table (if defined)

## Configuration:

Common variables used by this role:

| Variable | Description | Example |
|---|---|---|
| **aws_region** | AWS region | See [AWS regions](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) |
| **aws_access_key** | AWS access key | AKIAIOSFODNN7EXAMPLE |
| **aws_secret_key** | AWS secret key | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |
| **resource_tags** | List of project tags | see usage |

Variables specific to this role:

| Variable | Description | Example |
|---|---|---|
| **aws_route_tables** | List of Route Tables to create | see usage |
| **aws_route_tables.aws_rt_routes.target** | Route target type (Internet Gateway, NAT Gateway or Peering Connection | `igw`, `nat` or `vpc-peer` |
| **rt_tag** | The `Name:` tag of the Route Table to get information about | see usage |

## Usage:

How to invoke the role from a playbook:

```yaml
- name: Create AWS Route Tables
  include_role:
    name: aws_manage_route_tables
  vars:
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London
    resource_tags:
      Name: "{{ resource_name }}"   # Set to upper case aws_rt_name
      Owner: "Acme Co."

    aws_route_tables:
      - aws_rt_name: "rt-test-internet-a"
        aws_rt_vpc_tag: "Test A"
        aws_rt_region: "{{ aws_region }}"
        aws_rt_routes:
          - dest: "10.20.5.0/16"
            target: "peering"
            target_tag: "Test A-B"
          - dest: "0.0.0.0/0"
            target: "igw"
            target_tag: "Test A-IGW"
        aws_rt_subnets:
          - "Test-Public-A"
          - "Test-Public-B"
          - "Test-Public-C"
```

How to get Route Table facts:

```yaml
- name: Get Route Table facts
  include_role:
    name: aws_manage_route_tables
    tasks_from: "get_route_table_facts"
  vars:
    rt_tag: "rt-test-internet-a"
```

This will return facts in the variable `rt_facts`.  The data returned is as follows:

```yaml
"rt_facts": {
    "changed": false, 
    "failed": false, 
    "route_tables": [
        {
            "associations": [
                {
                    "id": "rtbassoc-50580438", 
                    "main": false, 
                    "route_table_id": "rtb-fbd46a93", 
                    "subnet_id": "subnet-a304f1d9"
                }, 
                {
                    "id": "rtbassoc-2d5f0345", 
                    "main": false, 
                    "route_table_id": "rtb-fbd46a93", 
                    "subnet_id": "subnet-315f657c"
                }, 
                {
                    "id": "rtbassoc-2e5f0346", 
                    "main": false, 
                    "route_table_id": "rtb-fbd46a93", 
                    "subnet_id": "subnet-c1db6aa8"
                }
            ], 
            "id": "rtb-fbd46a93", 
            "routes": [
                {
                    "destination_cidr_block": "10.10.0.0/22", 
                    "gateway_id": "local", 
                    "instance_id": null, 
                    "interface_id": null, 
                    "origin": "CreateRouteTable", 
                    "state": "active", 
                    "vpc_peering_connection_id": null
                }, 
                {
                    "destination_cidr_block": "10.20.0.0/22", 
                    "gateway_id": "local", 
                    "instance_id": null, 
                    "interface_id": null, 
                    "origin": "CreateRouteTable", 
                    "state": "active", 
                    "vpc_peering_connection_id": null
                }, 
                {
                    "destination_cidr_block": "10.30.0.0/22", 
                    "gateway_id": "local", 
                    "instance_id": null, 
                    "interface_id": null, 
                    "origin": "CreateRouteTable", 
                    "state": "active", 
                    "vpc_peering_connection_id": null
                }, 
                {
                    "destination_cidr_block": "0.0.0.0/0", 
                    "gateway_id": "igw-00cce969", 
                    "instance_id": null, 
                    "interface_id": null, 
                    "origin": "CreateRoute", 
                    "state": "active", 
                    "vpc_peering_connection_id": null
                }
            ], 
            "tags": {
                "Name": "TEST-CTRL-INTERNET", 
                "Owner": "Acme Co.", 
                "Project": "My Project"
            }, 
            "vpc_id": "vpc-ae0adfc6"
        }
    ]
}
```