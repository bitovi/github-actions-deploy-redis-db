# Deploy Redis Database (In Cluster Group)
`bitovi/github-actions-deploy-redis-db` deploys a Redis database cluster group. From one to multiple nodes.

This action uses the new GitHub Actions Commons, that is used by many Bitovi GitHub Actions, and so it's constantly evolving and improving.
![alt](https://bitovi-gha-pixel-tracker-deployment-main.bitovi-sandbox.com/pixel/9VGYuQniViBLOdi4tcBYU)
## Action Summary
As is, this action will deploy a single node (customizable) cluster group of Redis. 

If you would like to deploy a backend app/service, check out our other actions:
| Action | Purpose |
| ------ | ------- |
| [Deploy Docker to EC2](https://github.com/marketplace/actions/deploy-docker-to-aws-ec2) | Deploys a repo with a Dockerized application to a virtual machine (EC2) on AWS |
| [Deploy React to GitHub Pages](https://github.com/marketplace/actions/deploy-react-to-github-pages) | Builds and deploys a React application to GitHub Pages. |
| [Deploy static site to AWS (S3/CDN/R53)](https://github.com/marketplace/actions/deploy-static-site-to-aws-s3-cdn-r53) | Hosts a static site in AWS S3 with CloudFront |
<br/>

**And more!**, check our [list of actions in the GitHub marketplace](https://github.com/marketplace?category=&type=actions&verification=&query=bitovi)

# Need help or have questions?
This project is supported by [Bitovi, A DevOps consultancy](https://www.bitovi.com/services/devops-consulting).

You can **get help or ask questions** on our:

- [Discord Community](https://discord.gg/zAHn4JBVcX)


Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/services/devops-consulting).

# Basic Use

> **Note: ** Be sure to [set up your project for actions deployed pages](#set-up-your-project-for-actions-deployed-pages).

For basic usage, create `.github/workflows/deploy.yaml` with the following to build on push.

### Basic example
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Create an Redis DB
      id: create-redis
      uses: bitovi/github-actions-deploy-redis-db@v0.1.1
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

        aws_redis_enable: true
```

### Advanced example - Cluster enabled
```yaml
on:
  push:
    branches:
      - "main" # change to the branch you wish to deploy from

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Create an Redis DB
      id: create-redis
      uses: bitovi/github-actions-deploy-redis-db@v0.1.1
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_default_region: us-east-1

        aws_redis_enable: true

        aws_redis_num_cache_clusters: 0
        aws_redis_num_node_groups: 2
        aws_redis_replicas_per_node_group: 1
        aws_redis_parameter_group_name: default.redis7.cluster.on
        aws_redis_single_line_url_secret: true

        aws_redis_cloudwatch_log_type: slow-log,engine-log
```

## Customizing

> **Note: ** This action makes use of Terraform [elasticache_replication_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/elasticache_replication_group) resource. Most of the variables are mapped 1 to 1. Documentation on how to set it up could be found there.

### Inputs
1. [AWS](#aws-inputs)
1. [Action inputs](#action-inputs)
1. [Redis inputs](#redis-inputs)
1. [VPC Inputs](#vpc-inputs)

### Outputs
1. [Action Outputs](#action-outputs)

The following inputs can be used as `step.with` keys
<br/>
<br/>

#### **AWS Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_access_key_id` | String | AWS access key ID |
| `aws_secret_access_key` | String | AWS secret access key |
| `aws_session_token` | String | AWS session token |
| `aws_default_region` | String | AWS default region. Defaults to `us-east-1` |
| `aws_resource_identifier` | String | Set to override the AWS resource identifier for the deployment. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. Use with destroy to destroy specific resources. |
| `aws_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to all provisioned resources.|
<hr/>
<br/>

#### **Action Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `bitops_code_only` | Boolean | If `true`, will run only the generation phase of BitOps, where the Terraform and Ansible code is built. |
| `bitops_code_store` | Boolean | Store BitOps generated code as a GitHub artifact. |
| `tf_stack_destroy` | Boolean  | Set to `true` to destroy the stack - Will delete the `elb logs bucket` after the destroy action runs. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. Defaults to `tf-state-aws`, `tf-state-ecr` or `tf-state-eks.` |
| `tf_state_file_name_append` | String | Appends a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. (Can co-exist with `tf_state_file_name`) |
| `tf_state_bucket` | String | AWS S3 bucket name to use for Terraform state. See [note](#s3-buckets-naming) | 
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. Default is `false`. |
<hr/>
<br/>

#### **Redis Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_redis_enable` | Boolean | Enables the creation of a Redis instance. |
| `aws_redis_user` | String | Redis username. Defaults to `redisuser`. |
| `aws_redis_user_access_string` | String | String expression for user access. Defaults to `on ~* +@all`. |
| `aws_redis_user_group_name` | String | User group name. Defaults to `aws_resource_identifier-redis`. |
| `aws_redis_security_group_name` | String | Redis security group name. Defaults to `SG for aws_resource_identifier - Redis`. |
| `aws_redis_ingress_allow_all` | Boolean | Allow access from 0.0.0.0/0. Defaults to `true`. |
| `aws_redis_allowed_security_groups` | String | Comma separated list of security groups to be added to the Redis SG. |
| `aws_redis_subnets` | String | Define a list of specific subnets where Redis will live. Defaults to all of the VPC ones. If nome defined, default VPC. |
| `aws_redis_port` | String | Redis port. Defaults to `6379`. |
| `aws_redis_at_rest_encryption` | Boolean | Encryption at rest. Defaults to `true`. |
| `aws_redis_in_transit_encryption` | Boolean | In-transit encryption. Defaults to `true`. |
| `aws_redis_replication_group_id` | String | Name of the Redis replication group. Defaults to `aws_resource_identifier-redis`. |
| `aws_redis_node_type` | String | Node type of the Redis instance. Defaults to `cache.t2.small`. |
| `aws_redis_num_cache_clusters` | String | Amount of Redis nodes. Defaults to `1`. |
| `aws_redis_parameter_group_name` | String | Redis parameters groups name. If cluster wanted, set it to something that includes *.cluster.on.* Defaults to `default.redis7`. |
| `aws_redis_num_node_groups` | String | Number of node groups. Defaults to `0`. |
| `aws_redis_replicas_per_node_group` | String | Number of replicas per node group. Defaults to `0`. |
| `aws_redis_multi_az_enabled` | Boolean | Enables multi-availability-zone redis. Defaults to `false`. |
| `aws_redis_automatic_failover` | Boolean | Allows overriding the automatic configuration of this value, only needed when playing with resources in a non-conventional way. |
| `aws_redis_apply_immediately` | Boolean | Specifies whether any modifications are applied immediately, or during the next maintenance window. Defaults to `false`. |
| `aws_redis_auto_minor_upgrade` | Boolean | Specifies whether minor version engine upgrades will be applied automatically to the underlying Cache Cluster instances during the maintenance window. Defaults to `true`. |
| `aws_redis_maintenance_window` | String | Specifies the weekly time range for when maintenance on the cache cluster is performed. Example:sun:05:00-sun:06:00. Defaults to `null`. |
| `aws_redis_snapshot_window` | String | Daily time range (in UTC) when to start taking a daily snapshot. Minimum is a 60 minute period. Example: 05:00-09:00. Defaults to `null`. |
| `aws_redis_final_snapshot` | String | Change name to define a final snapshot. |
| `aws_redis_snapshot_restore_name` | String | Set name to restore a snapshot to the cluster. The default behaviour is to restore it each time this action runs. |
| `aws_redis_cloudwatch_enabled` | Boolean | Enable or disables Cloudwatch logging. |
| `aws_redis_cloudwatch_lg_name` | String | Cloudwatch log group name. Defaults to `/aws/redis/aws_resource_identifier` **Will append log_type to it** eg. `/your/name/slow-log`. |
| `aws_redis_cloudwatch_log_format` | String | Define log format between `json`(default) and text. |
| `aws_redis_cloudwatch_log_type` | String | Log type. Older Redis engines need `slow-log`. Newer support `engine-log` (default). You could add both by setting `slow-log,engine-log`.  |
| `aws_redis_cloudwatch_retention_days` | String | Number of days to retain cloudwatch logs. Defaults to `14`. |
| `aws_redis_single_line_url_secret`| Boolean | Creates an AWS secret containing the connection string `protocol://user@pass:endpoint:port`. |
| `aws_redis_additional_tags` | String | Additional tags to be added to every Redis related resource. |
<hr/>
<br/>

#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created |
| `aws_vpc_name` | String | Define a name for the VPC. Defaults to `VPC for ${aws_resource_identifier}`. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`|
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If no input, no private subnet will be created. Defaults to `<none>`. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region+<random>` value. If a list is defined, the first zone will be the one used for the EC2 instance. |
| `aws_vpc_id` | String | AWS VPC ID. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | AWS VPC Subnet ID. If none provided, will pick one. (Ideal when there's only one) |
| `aws_vpc_enable_nat_gateway` | Boolean | Adds a NAT gateway for each public subnet. Defaults to `false`. |
| `aws_vpc_single_nat_gateway` | Boolean | Toggles only one NAT gateway for all of the public subnets. Defaults to `false`. |
| `aws_vpc_external_nat_ip_ids` | String | **Existing** comma separated list of IP IDs if reusing. (ElasticIPs). |
| `aws_vpc_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to vpc provisioned resources.|
<hr/>
<br/>

#### **Action Outputs**
| Name             | Description                        |
|------------------|------------------------------------|
| `aws_vpc_id` | The selected VPC ID used. |
| `redis_endpoint` | Redis Endpoint. |
| `redis_secret_name` | Redis Secret name. |
| `redis_connection_string_secret` | Redis secret containing complete URL to connect directly. (e.g. rediss://user:pass@host:port). |
| `redis_sg_id` | Redis SG ID. |
<hr/>
<br/>


## Note about num_cache_clusters and num_node_groups

If you want to adjust num_node_groups and replicas_per_node_group, you **must** set **num_cache_clusters to 0**.
More about that can be found in the Terraform resource documentation.

## Note about resource identifiers

Most resources will contain the tag `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`, some of them, even the resource name after. 
We limit this to a 60 characters string because some AWS resources have a length limit and short it if needed.

We use the kubernetes style for this. For example, kubernetes -> k(# of characters)s -> k8s. And so you might see some compressions are made.

For some specific resources, we have a 32 characters limit. If the identifier length exceeds this number after compression, we remove the middle part and replace it for a hash made up from the string itself. 

## Note about tagging

There's the option to add any kind of defined tag's to each grouping module. Will be added to the commons tagging.
An example of how to set them: `{"key1": "value1", "key2": "value2"}`'

### S3 buckets naming

Buckets names can be made of up to 63 characters. If the length allows us to add -tf-state, we will do so. If not, a simple -tf will be added.

## Contributing
We would love for you to contribute to [`bitovi/github-actions-deploy-redis-db`](hhttps://github.com/bitovi/github-actions-deploy-redis-db).   [Issues](https://github.com/bitovi/github-actions-deploy-redis-db/issues) and [Pull Requests](https://github.com/bitovi/github-actions-deploy-redis-db/pulls) are welcome!

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/bitovi/github-actions-deploy-redis-db/blob/main/LICENSE).

# Provided by Bitovi
[Bitovi](https://www.bitovi.com/) is a proud supporter of Open Source software.

# We want to hear from you.
Come chat with us about open source in our Bitovi community [Discord](https://discord.gg/zAHn4JBVcX)!
