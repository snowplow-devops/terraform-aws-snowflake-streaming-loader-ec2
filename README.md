[![Release][release-image]][release] [![CI][ci-image]][ci] [![License][license-image]][license] [![Registry][registry-image]][registry] [![Source][source-image]][source]

# terraform-aws-snowflake-streaming-loader-ec2

A Terraform module which deploys a Snowplow Snowflake Streaming Loader application on AWS running on top of EC2.  If you want to use a custom AMI for this deployment you will need to ensure it is based on top of Amazon Linux 2.

## Telemetry

This module by default collects and forwards telemetry information to Snowplow to understand how our applications are being used.  No identifying information about your sub-account or account fingerprints are ever forwarded to us - it is very simple information about what modules and applications are deployed and active.

If you wish to subscribe to our mailing list for updates to these modules or security advisories please set the `user_provided_id` variable to include a valid email address which we can reach you at.

### How do I disable it?

To disable telemetry simply set variable `telemetry_enabled = false`.

### What are you collecting?

For details on what information is collected please see this module: https://github.com/snowplow-devops/terraform-snowplow-telemetry

## Usage

The Snowflake Streaming Loader reads data from a Snowplow Enriched output stream and writes in realtime to an events table.

```hcl
module "enriched_stream" {
  source  = "snowplow-devops/kinesis-stream/aws"
  version = "0.2.0"

  name = "enriched-stream"
}

module "bad_1_stream" {
  source  = "snowplow-devops/kinesis-stream/aws"
  version = "0.2.0"

  name = "bad-1-stream"
}

module "snowflake_streaming_loader" {
  source = "snowplow-devops/snowflake-streaming-loader-ec2/aws"

  accept_limited_use_license = true

  name                 = "sf-streaming-loader"
  vpc_id               = var.vpc_id
  subnet_ids           = var.subnet_ids
  in_stream_name       = module.enriched_stream.name
  bad_stream_name      = module.bad_1_stream.name

  snowflake_account_url = "<ACCOUNT_URL>"
  snowflake_database    = "<DATABASE>"
  snowflake_schema      = "<SCHEMA>"
  snowflake_loader_user = "<USER>"
  snowflake_loader_role = "<ROLE>"
  snowflake_private_key = "<PRIVATE_KEY>"

  ssh_key_name        = "your-key-name"
  ssh_ip_allowlist    = ["0.0.0.0/0"]
  health_ip_allowlist = ["0.0.0.0/0"]

  webhook_endpoint  = ""
  webhook_heartbeat = "5 minutes"

  skip_schemas = [
    "iglu:com.acme/skipped1/jsonschema/1-0-0",
    "iglu:com.acme/skipped2/jsonschema/1-0-*",
    "iglu:com.acme/skipped3/jsonschema/1-*-*",
    "iglu:com.acme/skipped4/jsonschema/*-*-*"
  ]
}
```

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 3.72.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 3.72.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_instance_type_metrics"></a> [instance\_type\_metrics](#module\_instance\_type\_metrics) | snowplow-devops/ec2-instance-type-metrics/aws | 0.1.2 |
| <a name="module_kcl_autoscaling"></a> [kcl\_autoscaling](#module\_kcl\_autoscaling) | snowplow-devops/dynamodb-autoscaling/aws | 0.2.0 |
| <a name="module_service"></a> [service](#module\_service) | snowplow-devops/service-ec2/aws | 0.3.2 |
| <a name="module_telemetry"></a> [telemetry](#module\_telemetry) | snowplow-devops/telemetry/snowplow | 0.6.0 |

## Resources

| Name | Type |
|------|------|
| [aws_cloudwatch_log_group.log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group) | resource |
| [aws_dynamodb_table.kcl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/dynamodb_table) | resource |
| [aws_iam_instance_profile.instance_profile](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile) | resource |
| [aws_iam_policy.iam_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy) | resource |
| [aws_iam_role.iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role_policy_attachment.policy_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource |
| [aws_security_group.sg](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) | resource |
| [aws_security_group_rule.egress_tcp_443](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.egress_tcp_80](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.egress_udp_123](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.ingress_tcp_22](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_security_group_rule.ingress_tcp_health](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule) | resource |
| [aws_caller_identity.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) | data source |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_bad_stream_name"></a> [bad\_stream\_name](#input\_bad\_stream\_name) | The name of the output kinesis stream that the loader will push bad data to | `string` | n/a | yes |
| <a name="input_in_stream_name"></a> [in\_stream\_name](#input\_in\_stream\_name) | The name of the input kinesis stream that the loader will pull data from | `string` | n/a | yes |
| <a name="input_name"></a> [name](#input\_name) | A name which will be pre-pended to the resources created | `string` | n/a | yes |
| <a name="input_snowflake_account_url"></a> [snowflake\_account\_url](#input\_snowflake\_account\_url) | URL for the connection to your Snowflake account | `string` | n/a | yes |
| <a name="input_snowflake_database"></a> [snowflake\_database](#input\_snowflake\_database) | Snowflake database name | `string` | n/a | yes |
| <a name="input_snowflake_loader_user"></a> [snowflake\_loader\_user](#input\_snowflake\_loader\_user) | Snowflake username used by loader to perform loading | `string` | n/a | yes |
| <a name="input_snowflake_private_key"></a> [snowflake\_private\_key](#input\_snowflake\_private\_key) | Private Key for snowflake\_loader\_user used by loader to perform loading | `string` | n/a | yes |
| <a name="input_snowflake_schema"></a> [snowflake\_schema](#input\_snowflake\_schema) | Snowflake schema name | `string` | n/a | yes |
| <a name="input_ssh_key_name"></a> [ssh\_key\_name](#input\_ssh\_key\_name) | The name of the SSH key-pair to attach to all EC2 nodes deployed | `string` | n/a | yes |
| <a name="input_subnet_ids"></a> [subnet\_ids](#input\_subnet\_ids) | The list of subnets to deploy the loader across | `list(string)` | n/a | yes |
| <a name="input_vpc_id"></a> [vpc\_id](#input\_vpc\_id) | The VPC to deploy the loader within | `string` | n/a | yes |
| <a name="input_accept_limited_use_license"></a> [accept\_limited\_use\_license](#input\_accept\_limited\_use\_license) | Acceptance of the SLULA terms (https://docs.snowplow.io/limited-use-license-1.0/) | `bool` | `false` | no |
| <a name="input_amazon_linux_2023_ami_id"></a> [amazon\_linux\_2023\_ami\_id](#input\_amazon\_linux\_2023\_ami\_id) | The AMI ID to use which must be based of of Amazon Linux 2023; by default the latest community version is used | `string` | `""` | no |
| <a name="input_app_version"></a> [app\_version](#input\_app\_version) | App version to use. This variable facilitates dev flow, the modules may not work with anything other than the default value. | `string` | `"0.5.1"` | no |
| <a name="input_associate_public_ip_address"></a> [associate\_public\_ip\_address](#input\_associate\_public\_ip\_address) | Whether to assign a public ip address to this instance | `bool` | `true` | no |
| <a name="input_cloudwatch_logs_enabled"></a> [cloudwatch\_logs\_enabled](#input\_cloudwatch\_logs\_enabled) | Whether application logs should be reported to CloudWatch | `bool` | `true` | no |
| <a name="input_cloudwatch_logs_retention_days"></a> [cloudwatch\_logs\_retention\_days](#input\_cloudwatch\_logs\_retention\_days) | The length of time in days to retain logs for | `number` | `7` | no |
| <a name="input_enable_auto_scaling"></a> [enable\_auto\_scaling](#input\_enable\_auto\_scaling) | Whether to enable auto-scaling policies for the service (WARN: ensure you have sufficient db\_connections available for the max number of nodes in the ASG) | `bool` | `true` | no |
| <a name="input_health_ip_allowlist"></a> [health\_ip\_allowlist](#input\_health\_ip\_allowlist) | The list of CIDR ranges to allow traffic from to health endpoint | `list(any)` | <pre>[<br/>  "0.0.0.0/0"<br/>]</pre> | no |
| <a name="input_iam_permissions_boundary"></a> [iam\_permissions\_boundary](#input\_iam\_permissions\_boundary) | The permissions boundary ARN to set on IAM roles created | `string` | `""` | no |
| <a name="input_initial_position"></a> [initial\_position](#input\_initial\_position) | Where to start processing the input Kinesis Stream from (TRIM\_HORIZON or LATEST) | `string` | `"TRIM_HORIZON"` | no |
| <a name="input_instance_type"></a> [instance\_type](#input\_instance\_type) | The instance type to use | `string` | `"t3a.small"` | no |
| <a name="input_java_opts"></a> [java\_opts](#input\_java\_opts) | Custom JAVA Options | `string` | `"-XX:InitialRAMPercentage=75 -XX:MaxRAMPercentage=75"` | no |
| <a name="input_kcl_read_max_capacity"></a> [kcl\_read\_max\_capacity](#input\_kcl\_read\_max\_capacity) | The maximum READ capacity for the KCL DynamoDB table | `number` | `10` | no |
| <a name="input_kcl_read_min_capacity"></a> [kcl\_read\_min\_capacity](#input\_kcl\_read\_min\_capacity) | The minimum READ capacity for the KCL DynamoDB table | `number` | `1` | no |
| <a name="input_kcl_write_max_capacity"></a> [kcl\_write\_max\_capacity](#input\_kcl\_write\_max\_capacity) | The maximum WRITE capacity for the KCL DynamoDB table | `number` | `10` | no |
| <a name="input_kcl_write_min_capacity"></a> [kcl\_write\_min\_capacity](#input\_kcl\_write\_min\_capacity) | The minimum WRITE capacity for the KCL DynamoDB table | `number` | `1` | no |
| <a name="input_max_size"></a> [max\_size](#input\_max\_size) | The maximum number of servers in this server-group | `number` | `2` | no |
| <a name="input_min_size"></a> [min\_size](#input\_min\_size) | The minimum number of servers in this server-group | `number` | `1` | no |
| <a name="input_private_ecr_registry"></a> [private\_ecr\_registry](#input\_private\_ecr\_registry) | The URL of an ECR registry that the sub-account has access to (e.g. '000000000000.dkr.ecr.cn-north-1.amazonaws.com.cn/') | `string` | `""` | no |
| <a name="input_scale_down_cooldown_sec"></a> [scale\_down\_cooldown\_sec](#input\_scale\_down\_cooldown\_sec) | Time (in seconds) until another scale-down action can occur | `number` | `600` | no |
| <a name="input_scale_down_cpu_threshold_percentage"></a> [scale\_down\_cpu\_threshold\_percentage](#input\_scale\_down\_cpu\_threshold\_percentage) | The average CPU percentage that we must be below to scale-down | `number` | `20` | no |
| <a name="input_scale_down_eval_minutes"></a> [scale\_down\_eval\_minutes](#input\_scale\_down\_eval\_minutes) | The number of consecutive minutes that we must be below the threshold to scale-down | `number` | `60` | no |
| <a name="input_scale_up_cooldown_sec"></a> [scale\_up\_cooldown\_sec](#input\_scale\_up\_cooldown\_sec) | Time (in seconds) until another scale-up action can occur | `number` | `180` | no |
| <a name="input_scale_up_cpu_threshold_percentage"></a> [scale\_up\_cpu\_threshold\_percentage](#input\_scale\_up\_cpu\_threshold\_percentage) | The average CPU percentage that must be exceeded to scale-up | `number` | `60` | no |
| <a name="input_scale_up_eval_minutes"></a> [scale\_up\_eval\_minutes](#input\_scale\_up\_eval\_minutes) | The number of consecutive minutes that the threshold must be breached to scale-up | `number` | `5` | no |
| <a name="input_skip_schemas"></a> [skip\_schemas](#input\_skip\_schemas) | A list of schemas that won't be loaded to Snowflake | `list(string)` | `[]` | no |
| <a name="input_snowflake_loader_role"></a> [snowflake\_loader\_role](#input\_snowflake\_loader\_role) | Snowflake role used by loader to perform loading | `string` | `""` | no |
| <a name="input_ssh_ip_allowlist"></a> [ssh\_ip\_allowlist](#input\_ssh\_ip\_allowlist) | The list of CIDR ranges to allow SSH traffic from | `list(any)` | <pre>[<br/>  "0.0.0.0/0"<br/>]</pre> | no |
| <a name="input_tags"></a> [tags](#input\_tags) | The tags to append to this resource | `map(string)` | `{}` | no |
| <a name="input_telemetry_enabled"></a> [telemetry\_enabled](#input\_telemetry\_enabled) | Whether or not to send telemetry information back to Snowplow Analytics Ltd | `bool` | `true` | no |
| <a name="input_user_provided_id"></a> [user\_provided\_id](#input\_user\_provided\_id) | An optional unique identifier to identify the telemetry events emitted by this stack | `string` | `""` | no |
| <a name="input_webhook_endpoint"></a> [webhook\_endpoint](#input\_webhook\_endpoint) | HTTP endpoint to report monitoring alerts and heartbeats | `string` | `""` | no |
| <a name="input_webhook_heartbeat"></a> [webhook\_heartbeat](#input\_webhook\_heartbeat) | How often to send the heartbeat event | `string` | `"5 minutes"` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_asg_id"></a> [asg\_id](#output\_asg\_id) | ID of the ASG |
| <a name="output_asg_name"></a> [asg\_name](#output\_asg\_name) | Name of the ASG |
| <a name="output_sg_id"></a> [sg\_id](#output\_sg\_id) | ID of the security group attached to the servers |

# Copyright and license

Copyright 2025-present Snowplow Analytics Ltd.

Licensed under the [Snowplow Limited Use License Agreement][license]. _(If you are uncertain how it applies to your use case, check our answers to [frequently asked questions][license-faq].)_

[release]: https://github.com/snowplow-devops/terraform-aws-snowflake-streaming-loader-ec2/releases/latest
[release-image]: https://img.shields.io/github/v/release/snowplow-devops/terraform-aws-snowflake-streaming-loader-ec2

[ci]: https://github.com/snowplow-devops/terraform-aws-snowflake-streaming-loader-ec2/actions?query=workflow%3Aci
[ci-image]: https://github.com/snowplow-devops/terraform-aws-snowflake-streaming-loader-ec2/workflows/ci/badge.svg

[license]: https://docs.snowplow.io/limited-use-license-1.1/
[license-image]: https://img.shields.io/badge/license-Snowplow--Limited--Use-blue.svg?style=flat
[license-faq]: https://docs.snowplow.io/docs/contributing/limited-use-license-faq/

[registry]: https://registry.terraform.io/modules/snowplow-devops/snowflake-streaming-loader-ec2/aws/latest
[registry-image]: https://img.shields.io/static/v1?label=Terraform&message=Registry&color=7B42BC&logo=terraform

[source]: https://github.com/snowplow-incubator/snowflake-loader
[source-image]: https://img.shields.io/static/v1?label=Snowplow&message=Snowflake%20Loader&color=0E9BA4&logo=GitHub
