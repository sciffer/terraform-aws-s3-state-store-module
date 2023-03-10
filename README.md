<!-- BEGIN_TF_DOCS -->
# S3 Terrform State Storage Terraform Module

A basic S3 Terraform state sotrage module based on best practices and meant mostly to save time and husstle of figuring out how to set it up.

## Description

After creayting mutliple storages I thought it would be a good idea to wrap it in a module that wil be reusable and everyone could save the time and hustle of a "battle tested" setup.
The bucket is enabling by default versioning, deletion protection, implicitly blocking public access and encryption(to protect the secrets that can end up on he state files), in addition it will provision a dynamoDB table for locking mechanism(based on best practices).

Would love any input or contribution to that setup and hope someone else will be able to enjoy such a trivial module.

## Getting Started

### Initial setup of the storage resources

Importnt: It is recommended that you setup a dedicated repo just for the state storage, you can follow the bellow guidelines to setup the state store:
1. Setup a new repo for holding the state storage terraform setup only and initialize the local files using 'terraform init'.
2. Prepare a basic providers file with local storage first(used to setup the state resources). Here is an example(replace the relevant fields with your specifics):
'''
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}
provider "aws" {
  region = "us-east-1"
}
'''
3. Prepare your storage definition, like the bellow example. Please add/replace relevant fields with the settings that suite your needs, settings description can be found at the inputs table bellow:
'''
module "s3_state_store" {
  source = "sciffer/s3-state-store-module/aws"
  bucket_name = "terraform_state"
  dynamodb_table_name = "terraform_state_locks"
}
'''
4. Create the bucket and dynamoDB table, make sure the resources match your selections in the plan output:
'''
$ terraform plan
$ terraform apply
'''

### Migrating the local state to the newly created S3 state storage

Follow the bellow steps to migrate the state from local to the new S3 state storage:
1. Update the providers file with the following backend block, please adjust the bucket name, state key(should reflect a unique path to this terraform state file), region and dynamodb table name. The end result should follow the bellow format:
'''
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }

  backend "s3" {
    bucket = "terraform-state"
    key = "global/s3-state-storge/terraform.tfstate"
    region = "us-east-1"
    dynamodb_table = "terraform_state_locks"
    encrypt = true
  }
}
provider "aws" {
    region = "us-east-1"
}
'''
2. Reinitialize the state to migrate it to S3 by issuing: 'terraform init -migrate-state' and approving the migration if everything seems correct.
3. Rerun plan and apply to verify the state is properly functioning.

### Adding additional terraform setups to use the S3 storage

To add an exisitng or new terraform projects to the defined storage above you'll need the following:
* Add the same backend block to the terraform resource in the providers file, but make sure to change the 'key' value and make sure it is unique for each terraform project.
* The same procedure that is defined above can be used to migrate any existing terraform projects to the S3 bucket.

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider_aws) | n/a |

## Inputs

| Name | Description | Type |
|------|-------------|------|
| <a name="input_bucket_name"></a> [bucket_name](#input_bucket_name) | Mandatory: Name of the S3 state bucket | `string` |
| <a name="input_dynamodb_table_name"></a> [dynamodb_table_name](#input_dynamodb_table_name) | Mandatory: Dynamo lock table name | `string` |
| <a name="input_block_public_access"></a> [block_public_access](#input_block_public_access) | Optional: Implicitly block public access(Defaults to true) | `bool` |
| <a name="input_dynamodb_billing_mode"></a> [dynamodb_billing_mode](#input_dynamodb_billing_mode) | Optional: Dynamo table billing mode(Defaults to PAY_PER_REQUEST) | `string` |
| <a name="input_sse_algorithm"></a> [sse_algorithm](#input_sse_algorithm) | Optional: Encryption algorithm used to protect the state(Defaults to AES256) | `string` |
| <a name="input_versioning"></a> [versioning](#input_versioning) | Optional: Enable S3 versioning(Defaults to true) | `bool` |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_bucket_name"></a> [bucket_name](#output_bucket_name) | The bucket name used for state storage. |
| <a name="output_bucket_region"></a> [bucket_region](#output_bucket_region) | The bucket region used for state storage. |
| <a name="output_dynamodb_table_name"></a> [dynamodb_table_name](#output_dynamodb_table_name) | The dynamodb table name used for state lock. |

## Authors

Contributors names and contact info

Shalom Cohen (https://github.com/sciffer)

## License

This project is licensed under the MIT License - see the LICENSE.md file for details
<!-- END_TF_DOCS -->