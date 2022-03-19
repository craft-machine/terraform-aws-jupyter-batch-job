# Jupyter Batch Job module

## TODO

- [PLT-495](https://craftmachine.atlassian.net/browse/PLT-495)
- [PLT-496](https://craftmachine.atlassian.net/browse/PLT-496)

## Usage Example:

```
module "my_job" {
  source = "git::https://bitbucket.org/craftmachine/aws-modules//jupyter-batch-job?ref=428" # Put ref tag corresponding to the version you need

  notebook_relative_path   = "dir/notebook-for-my-job.ipynb" # notebook path, relative to s3 root path where your notebooks are being uploaded
  notebook_repository_name = "my-awesome-reposiory"      # dash-case
  notebook_group_name      = "etl"                       # snake_case, for anything related to data movement put "etl" here

  # Parameters passed to the Notebook via Papermill
  notebook_parameters = {
    ENV              = "${terraform.workspace}"
    DATE_START       = "2019-01-01"
    DATE_END         = "2019-01-02"
    ONE_MORE_PARAM   = "Some Value"
  }

  notebook_execution_name_pattern = "{notebook.name}/{execution.timestamp}"

  container_vcpus  = 1
  container_memory = 1800 # MiB
  job_timeout      = 7200 # Seconds
  job_queue_arn    = "${data.terraform_remote_state.network.batch_etl_internal_sequential_job_queue_arn}"

  schedule_name                     = "daily"       # (Optional) Name of schedule. Will be used to generate Cloudwatch rule name and Batch Job name. Good values are \"daily\", \"hourly\", \"periodic\", etc.
  schedule_cron                     = "0 2 * * ? *" # (Optional) Remove this line if you don't need scheduling


  notebooks_s3_bucket_name = "${data.terraform_remote_state.bi_network.jupyter_notebooks_s3_bucket_name}"
  env                      = "${terraform.workspace}"
}
```

If your notebook needs additional permissions, you can use `job_role_name` output variable from the module to attach your own policies to the role.

Example:

```
resource "aws_iam_role_policy" "my_policy" {
  name   = "my_policy"
  role   = "${module.my_job.job_role_name}" # Get role name from module
  policy = "${data.aws_iam_policy_document.my_policy.json}"
}

data "aws_iam_policy_document" "my_policy" {
  statement {
    ...
  }
}
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requirements

No requirements.

## Providers

| Name | Version |
| ---- | ------- |
| aws  | n/a     |

## Inputs

| Name                               | Description                                                                                                                                                                                                                                                                           | Type          | Default | Required |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------- | :------: |
| capacity_provider                  | Name of the capacity provider                                                                                                                                                                                                                                                         | `string`      | `""`    |    no    |
| cluster_id                         | ARN of an ECS cluster.                                                                                                                                                                                                                                                                | `string`      | n/a     |   yes    |
| container_definitions              | The ECS task definition data source.                                                                                                                                                                                                                                                  | `string`      | n/a     |   yes    |
| container_port                     | The port on the container to associate with the load balancer.                                                                                                                                                                                                                        | `number`      | n/a     |   yes    |
| deployment_maximum_percent         | The upper limit (as a percentage of the service's desiredCount) of the number of running tasks that can be running in a service during a deployment.                                                                                                                                  | `number`      | `200`   |    no    |
| deployment_minimum_healthy_percent | The lower limit (as a percentage of the service's desiredCount) of the number of running tasks that must remain running and healthy in a service during a deployment.                                                                                                                 | `number`      | `50`    |    no    |
| desired_count                      | The number of instances of the task definition to place and keep running.                                                                                                                                                                                                             | `number`      | n/a     |   yes    |
| ecs_service_role_arn               | ARN of default Amazon ECS service role.                                                                                                                                                                                                                                               | `string`      | n/a     |   yes    |
| health_check                       | The destination for the health check request. Default /.                                                                                                                                                                                                                              | `string`      | `"/"`   |    no    |
| health_check_healthy_threshold     | The number of consecutive health checks successes required before considering an unhealthy target healthy.                                                                                                                                                                            | `number`      | `3`     |    no    |
| health_check_interval              | The approximate amount of time, in seconds, between health checks of an individual target                                                                                                                                                                                             | `number`      | `30`    |    no    |
| health_check_timeout               | The amount of time, in seconds, during which no response means a failed health check.                                                                                                                                                                                                 | `number`      | `5`     |    no    |
| health_check_unhealthy_threshold   | The number of consecutive health check failures required before considering the target unhealthy.                                                                                                                                                                                     | `number`      | `3`     |    no    |
| name                               | Name to be used on all the resources as identifier                                                                                                                                                                                                                                    | `string`      | `""`    |    no    |
| tags                               | A map of tags to add to all resources                                                                                                                                                                                                                                                 | `map(string)` | `{}`    |    no    |
| volumes                            | A list of volumes that containers in service task will have access to. List item structure should mirror volume argument of aws_ecs_task_definition resource: https://registry.terraform.io/providers/hashicorp/aws/3.27.0/docs/resources/ecs_task_definition#volume-block-arguments. | `any`         | `[]`    |    no    |
| vpc_id                             | VPC that will be used for all resources.                                                                                                                                                                                                                                              | `string`      | n/a     |   yes    |
| wait_for_steady_state              | If true, Terraform will wait for the service to reach a steady state                                                                                                                                                                                                                  | `bool`        | `false` |    no    |

## Outputs

| Name                         | Description |
| ---------------------------- | ----------- |
| ecs_task_execution_role_name | n/a         |
| ecs_task_role_name           | n/a         |
| lb_target_group_arn          | n/a         |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
