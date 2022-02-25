## Barracuda

This module is used by BNBTS projects, not React/API.


## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/cloudposse/terraform-aws-rds-cluster/releases).



[Basic example](examples/basic)

```hcl
module "rds_cluster_aurora_postgres" {
  source          = "git::https://github.com/cloudposse/terraform-aws-rds-cluster.git?ref=master"
  name            = "postgres"
  engine          = "aurora-postgresql"
  cluster_family  = "aurora-postgresql9.6"
  cluster_size    = "2"
  namespace       = "eg"
  stage           = "dev"
  admin_user      = "admin1"
  admin_password  = "Test123456789"
  db_name         = "dbname"
  db_port         = "5432"
  instance_type   = "db.r4.large"
  vpc_id          = "vpc-xxxxxxxx"
  security_groups = ["sg-xxxxxxxx"]
  subnets         = ["subnet-xxxxxxxx", "subnet-xxxxxxxx"]
  zone_id         = "Zxxxxxxxx"
}
```


[Serverless MySQL](examples/serverless_mysql)

```hcl
module "rds_cluster_aurora_mysql_serverless" {
  source          = "git::https://github.com/cloudposse/terraform-aws-rds-cluster.git?ref=master"
  namespace       = "eg"
  stage           = "dev"
  name            = "db"
  engine          = "aurora"
  engine_mode     = "serverless"
  cluster_family  = "aurora5.6"
  cluster_size    = "0"
  admin_user      = "admin1"
  admin_password  = "Test123456789"
  db_name         = "dbname"
  db_port         = "3306"
  instance_type   = "db.t2.small"
  vpc_id          = "vpc-xxxxxxxx"
  security_groups = ["sg-xxxxxxxx"]
  subnets         = ["subnet-xxxxxxxx", "subnet-xxxxxxxx"]
  zone_id         = "Zxxxxxxxx"

  scaling_configuration = [
    {
      auto_pause               = true
      max_capacity             = 256
      min_capacity             = 2
      seconds_until_auto_pause = 300
    }
  ]
}
```

[With cluster parameters](examples/with_cluster_parameters)

```hcl
module "rds_cluster_aurora_mysql" {
  source          = "git::https://github.com/cloudposse/terraform-aws-rds-cluster.git?ref=master"
  engine          = "aurora"
  cluster_family  = "aurora-mysql5.7"
  cluster_size    = "2"
  namespace       = "eg"
  stage           = "dev"
  name            = "db"
  admin_user      = "admin1"
  admin_password  = "Test123456789"
  db_name         = "dbname"
  instance_type   = "db.t2.small"
  vpc_id          = "vpc-xxxxxxx"
  security_groups = ["sg-xxxxxxxx"]
  subnets         = ["subnet-xxxxxxxx", "subnet-xxxxxxxx"]
  zone_id         = "Zxxxxxxxx"

  cluster_parameters = [
    {
      name  = "character_set_client"
      value = "utf8"
    },
    {
      name  = "character_set_connection"
      value = "utf8"
    },
    {
      name  = "character_set_database"
      value = "utf8"
    },
    {
      name  = "character_set_results"
      value = "utf8"
    },
    {
      name  = "character_set_server"
      value = "utf8"
    },
    {
      name  = "collation_connection"
      value = "utf8_bin"
    },
    {
      name  = "collation_server"
      value = "utf8_bin"
    },
    {
      name         = "lower_case_table_names"
      value        = "1"
      apply_method = "pending-reboot"
    },
    {
      name         = "skip-character-set-client-handshake"
      value        = "1"
      apply_method = "pending-reboot"
    }
  ]
}
```

[With enhanced monitoring](examples/enhanced_monitoring)

```hcl
# create IAM role for monitoring
resource "aws_iam_role" "enhanced_monitoring" {
  name               = "rds-cluster-example-1"
  assume_role_policy = data.aws_iam_policy_document.enhanced_monitoring.json
}

# Attach Amazon's managed policy for RDS enhanced monitoring
resource "aws_iam_role_policy_attachment" "enhanced_monitoring" {
  role       = aws_iam_role.enhanced_monitoring.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonRDSEnhancedMonitoringRole"
}

# allow rds to assume this role
data "aws_iam_policy_document" "enhanced_monitoring" {
  statement {
      actions = [
      "sts:AssumeRole",
    ]

    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["monitoring.rds.amazonaws.com"]
    }
  }
}

module "rds_cluster_aurora_postgres" {
  source          = "git::https://github.com/cloudposse/terraform-aws-rds-cluster.git?ref=master"
  engine          = "aurora-postgresql"
  cluster_family  = "aurora-postgresql9.6"
  cluster_size    = "2"
  namespace       = "eg"
  stage           = "dev"
  name            = "db"
  admin_user      = "admin1"
  admin_password  = "Test123456789"
  db_name         = "dbname"
  db_port         = "5432"
  instance_type   = "db.r4.large"
  vpc_id          = "vpc-xxxxxxx"
  security_groups = ["sg-xxxxxxxx"]
  subnets         = ["subnet-xxxxxxxx", "subnet-xxxxxxxx"]
  zone_id         = "Zxxxxxxxx"

  # enable monitoring every 30 seconds
  rds_monitoring_interval = 30

  # reference iam role created above
  rds_monitoring_role_arn = aws_iam_role.enhanced_monitoring.arn
}
```






## Makefile Targets
```
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen
  lint                                Lint terraform code

```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| admin_password | (Required unless a snapshot_identifier is provided) Password for the master DB user | string | `` | no |
| admin_user | (Required unless a snapshot_identifier is provided) Username for the master DB user | string | `admin` | no |
| allowed_cidr_blocks | List of CIDR blocks allowed to access the cluster | list(string) | `<list>` | no |
| apply_immediately | Specifies whether any cluster modifications are applied immediately, or during the next maintenance window | bool | `true` | no |
| attributes | Additional attributes (e.g. `1`) | list(string) | `<list>` | no |
| autoscaling_enabled | Whether to enable cluster autoscaling | bool | `false` | no |
| autoscaling_max_capacity | Maximum number of instances to be maintained by the autoscaler | number | `5` | no |
| autoscaling_min_capacity | Minimum number of instances to be maintained by the autoscaler | number | `1` | no |
| autoscaling_policy_type | Autoscaling policy type. `TargetTrackingScaling` and `StepScaling` are supported | string | `TargetTrackingScaling` | no |
| autoscaling_scale_in_cooldown | The amount of time, in seconds, after a scaling activity completes and before the next scaling down activity can start. Default is 300s | number | `300` | no |
| autoscaling_scale_out_cooldown | The amount of time, in seconds, after a scaling activity completes and before the next scaling up activity can start. Default is 300s | number | `300` | no |
| autoscaling_target_metrics | The metrics type to use. If this value isn't provided the default is CPU utilization | string | `RDSReaderAverageCPUUtilization` | no |
| autoscaling_target_value | The target value to scale with respect to target metrics | number | `75` | no |
| backtrack_window | The target backtrack window, in seconds. Only available for aurora engine currently. Must be between 0 and 259200 (72 hours) | number | `0` | no |
| backup_window | Daily time range during which the backups happen | string | `07:00-09:00` | no |
| cluster_dns_name | Name of the cluster CNAME record to create in the parent DNS zone specified by `zone_id`. If left empty, the name will be auto-asigned using the format `master.var.name` | string | `` | no |
| cluster_family | The family of the DB cluster parameter group | string | `aurora5.6` | no |
| cluster_parameters | List of DB cluster parameters to apply | object | `<list>` | no |
| cluster_size | Number of DB instances to create in the cluster | number | `2` | no |
| copy_tags_to_snapshot | Copy tags to backup snapshots | bool | `false` | no |
| db_name | Database name (default is not to create a database) | string | `` | no |
| db_port | Database port | number | `3306` | no |
| deletion_protection | If the DB instance should have deletion protection enabled | bool | `false` | no |
| delimiter | Delimiter to be used between `namespace`, `environment`, `stage`, `name` and `attributes` | string | `-` | no |
| enabled | Set to false to prevent the module from creating any resources | bool | `true` | no |
| enabled_cloudwatch_logs_exports | List of log types to export to cloudwatch. The following log types are supported: audit, error, general, slowquery | list(string) | `<list>` | no |
| engine | The name of the database engine to be used for this DB cluster. Valid values: `aurora`, `aurora-mysql`, `aurora-postgresql` | string | `aurora` | no |
| engine_mode | The database engine mode. Valid values: `parallelquery`, `provisioned`, `serverless` | string | `provisioned` | no |
| engine_version | The version of the database engine to use. See `aws rds describe-db-engine-versions` | string | `` | no |
| environment | Environment, e.g. 'prod', 'staging', 'dev', 'pre-prod', 'UAT' | string | `` | no |
| global_cluster_identifier | ID of the Aurora global cluster | string | `` | no |
| iam_database_authentication_enabled | Specifies whether or mappings of AWS Identity and Access Management (IAM) accounts to database accounts is enabled | bool | `false` | no |
| iam_roles | Iam roles for the Aurora cluster | list(string) | `<list>` | no |
| instance_availability_zone | Optional parameter to place cluster instances in a specific availability zone. If left empty, will place randomly | string | `` | no |
| instance_parameters | List of DB instance parameters to apply | object | `<list>` | no |
| instance_type | Instance type to use | string | `db.t2.small` | no |
| kms_key_arn | The ARN for the KMS encryption key. When specifying `kms_key_arn`, `storage_encrypted` needs to be set to `true` | string | `` | no |
| maintenance_window | Weekly time range during which system maintenance can occur, in UTC | string | `wed:03:00-wed:04:00` | no |
| name | Solution name, e.g. 'app' or 'jenkins' | string | `` | no |
| namespace | Namespace, which could be your organization name or abbreviation, e.g. 'eg' or 'cp' | string | `` | no |
| performance_insights_enabled | Whether to enable Performance Insights | bool | `false` | no |
| performance_insights_kms_key_id | The ARN for the KMS key to encrypt Performance Insights data. When specifying `performance_insights_kms_key_id`, `performance_insights_enabled` needs to be set to true | string | `` | no |
| publicly_accessible | Set to true if you want your cluster to be publicly accessible (such as via QuickSight) | bool | `false` | no |
| rds_monitoring_interval | Interval in seconds that metrics are collected, 0 to disable (values can only be 0, 1, 5, 10, 15, 30, 60) | number | `0` | no |
| rds_monitoring_role_arn | The ARN for the IAM role that can send monitoring metrics to CloudWatch Logs | string | `` | no |
| reader_dns_name | Name of the reader endpoint CNAME record to create in the parent DNS zone specified by `zone_id`. If left empty, the name will be auto-asigned using the format `replicas.var.name` | string | `` | no |
| replication_source_identifier | ARN of a source DB cluster or DB instance if this DB cluster is to be created as a Read Replica | string | `` | no |
| retention_period | Number of days to retain backups for | number | `5` | no |
| scaling_configuration | List of nested attributes with scaling properties. Only valid when `engine_mode` is set to `serverless` | object | `<list>` | no |
| security_groups | List of security groups to be allowed to connect to the DB instance | list(string) | `<list>` | no |
| skip_final_snapshot | Determines whether a final DB snapshot is created before the DB cluster is deleted | bool | `true` | no |
| snapshot_identifier | Specifies whether or not to create this cluster from a snapshot | string | `` | no |
| source_region | Source Region of primary cluster, needed when using encrypted storage and region replicas | string | `` | no |
| stage | Stage, e.g. 'prod', 'staging', 'dev', OR 'source', 'build', 'test', 'deploy', 'release' | string | `` | no |
| storage_encrypted | Specifies whether the DB cluster is encrypted. The default is `false` for `provisioned` `engine_mode` and `true` for `serverless` `engine_mode` | bool | `false` | no |
| subnets | List of VPC subnet IDs | list(string) | - | yes |
| tags | Additional tags (e.g. `map('BusinessUnit','XYZ')` | map(string) | `<map>` | no |
| timeouts_configuration | List of timeout values per action. Only valid actions are `create`, `update` and `delete` | object | `<list>` | no |
| vpc_id | VPC ID to create the cluster in (e.g. `vpc-a22222ee`) | string | - | yes |
| vpc_security_group_ids | Additional security group IDs to apply to the cluster, in addition to the provisioned default security group with ingress traffic from existing CIDR blocks and existing security groups | list(string) | `<list>` | no |
| zone_id | Route53 parent zone ID. If provided (not empty), the module will create sub-domain DNS records for the DB master and replicas | string | `` | no |

## Outputs

| Name | Description |
|------|-------------|
| arn | Amazon Resource Name (ARN) of cluster |
| cluster_identifier | Cluster Identifier |
| cluster_resource_id | The region-unique, immutable identifie of the cluster |
| cluster_security_groups | Default RDS cluster security groups |
| database_name | Database name |
| dbi_resource_ids | List of the region-unique, immutable identifiers for the DB instances in the cluster |
| endpoint | The DNS address of the RDS instance |
| master_host | DB Master hostname |
| master_username | Username for the master DB user |
| reader_endpoint | A read-only endpoint for the Aurora cluster, automatically load-balanced across replicas |
| replicas_host | Replicas hostname |



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
