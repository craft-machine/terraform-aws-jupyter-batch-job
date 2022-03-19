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
|------|---------|
| aws | n/a |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| aws\_region | AWS Region | `string` | `"us-east-1"` | no |
| container\_memory | Memory size in Megabytes allocated to container. | `number` | `1800` | no |
| container\_mount\_points | List of mount points for provided volumes. | `any` | `[]` | no |
| container\_vcpus | Number of VCPU allocated to container. | `number` | `1` | no |
| container\_volumes | List of volumes you need to mount. | `any` | `[]` | no |
| docker\_image | Docker image used for job. Image must contain installed papercraft. | `string` | `""` | no |
| env | Environment code, like "qa", "staging", "production" | `any` | n/a | yes |
| job\_queue\_arn | AWS Batch Job Queue ARN. | `any` | n/a | yes |
| job\_retry\_attempts | Number of retries per job attempt. | `number` | `1` | no |
| job\_timeout | Job timeout in seconds. This timeout is per attempt. | `number` | `3600` | no |
| notebook\_execution\_name\_pattern | Pattern used to save execution outputs to S3. The most commonly used one is "{notebook.name}/{execution.timestamp}" | `any` | n/a | yes |
| notebook\_group\_name | Name of top-level grouping for notebooks. Usually it is "etl". Other suggestions are "data\_science", "data\_monitoring", "reports" | `string` | `"etl"` | no |
| notebook\_parameters | Parameters passed to the notebook via Papermill. Must be entered as terraform "map" type. Under the hood is converted into json before passing to Papermill. | `map(string)` | `{}` | no |
| notebook\_relative\_path | Path to notebook inside repository folder that is copied to S3. | `any` | n/a | yes |
| notebook\_repository\_name | Name of git repository. Is used as a part of path to notebook location on S3. | `any` | n/a | yes |
| notebooks\_s3\_bucket\_name | S3 Bucket where jupyter notebooks and executions are stored. | `any` | n/a | yes |
| schedule\_cron | (Optional) Cron expression, e.g. "0 2 \* \* ? \*". If blank the schedule is not created. | `string` | `""` | no |
| schedule\_enabled | (Optional) To temporarily disable schedule, set this to "false". | `bool` | `true` | no |
| schedule\_name | (Optional) Name of schedule. Will be used to generate Cloudwatch rule name and Batch Job name. Good values are "daily", "hourly", "periodic", etc. | `string` | `"schedule"` | no |

## Outputs

| Name | Description |
|------|-------------|
| job\_definition\_name | n/a |
| job\_role\_arn | n/a |
| job\_role\_name | n/a |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
