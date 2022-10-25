# API Data Collector

## Things to be reminded 
1. In secret manager there's a field name isFirstTime 
2. The isFirstTime value should be true for the initial run, 
3. After initial run the value `must` be updated to false, to avoid duplicate query the data forever.

## Deploy in the following order

1. api-aws-iac
1. api-datacollector
1. api-dataretriever

## How to start

* Make sure you install `Terraform` and `git` in your computer
* Clone the project
    ```
    git clone https://gitlab.idorsia.com/gis-ieo/tbscg/api-datacollector.git
    ```
* Open source code in VSCode or any other Editor
* Open file `terraform/terraform.tfvars`
* Open file `deploy_infra.sh`

* Open file `terraform/variables.tf`. Line (6-15)
    * Update the
        * `env_name=`
        * `vpc_id=`
        * `subnet_id=`
        * `vpc_endpoint_id=`
    * If you want to deploy prod environment also then copy `env_1` and create `env_2` and update all fields as
      following
    ```shell
      variable "env_list" {
            default = {
              env_1 = {
                 env_name        = "test",
                 vpc_id          = "vpc-123456789"
                 subnet_id       = "subnet-123456789"
                 vpc_endpoint_id = "vpce-123456789"
              },
              env_2 = {
                 env_name        = "prod",
                 vpc_id          = "vpc-1234567890"
                 subnet_id       = "subnet-1234567890"  
                 vpc_endpoint_id = "vpce-1234567890"
              }
           }
      }
    ```
* You can create a new environment by copying same way and change the env name and others information

## Artifacts Generation

* Gitlab CI/CD Pipeline Settings
    * Please update the following variables in repository CI/CD Settings
        ```yaml
        AWS_S3_BUCKET
        ```
    * Type: Variable
    * Environment scope: all
    * Generally CI/CD settings are available at
      this [Link](https://gitlab.idorsia.com/gis-ieo/tbscg/api-dataretriever/-/settings/ci_cd)
* When you push a change in **main** branch then GitLab will run the CI/CD pipeline (`DataCollector/.gitlab-ci.yml`)
* This file will build the java code and create jar, and push the jar to **AWS S3**
* When you deploy the lambda, then these artifacts(jar) will copy to local and deploy in AWS Cloud

## To deploy the project
* Before deployment the jar file should move to {rood.dir}/terraform/lambda_jars/{file_name.jar}
    * This command can easily run using shell script (deploy_infra.sh)
* Open a Terminal in root project
* Run the following command in terminal
    * Mac and Linux
      ```shell
      ./deploy_infra.sh
      ```
    * Windows (on git bash)
        ```shell
        sh ./deploy_infra.sh
      ```
    * This will run all required steps related to terraform for deploy in AWS Cloud

## To destroy or uninstall the project

* Open a Terminal in root project
* Run the following command in terminal
    * Mac and Linux
      ```shell
      ./destroy_infra.sh
      ```
    * Windows (on git bash)
        ```shell
        sh ./destroy_infra.sh
      ```
    * This will run all required steps related to terraform for destroy in AWS Cloud
    * In some steps, you need to input `yes`

### API URL's

* ALL API url will be private by default
    * Under the `VPC ID`, `Subnet Id`, and `VPC Endpoint ID` you provide in `terraform/variables.tf` file
    * So, you can only access from any client under above VPC and Subnet.

* If you want to create new VPC endpoint (if not exists) under current VPC and Subnet
    * Open the file `terraform/modules/module_base/api_gateway_private.tf`
    * Comment Line 7
    * And Uncomment line 8
    * Open the file `terraform/modules/module_base/vpc_endpoint.tf`
    * Also Uncomment Line 1 to 10 for `module_vpc_endpoint`

* If you want to make all API urls to public and want to access from internet
    * Open the file `terraform/modules/module_base/api_gateway_public.tf`
    * Uncomment Line 1 to 13.
    * Open the file `terraform/modules/module_base/api_gateway_private.tf`
    * And Comment Line 1 to 27.

* All URL's will be published in terminal after successful deploy to AWS Cloud
* From the URL pattern you cannot detect the which are public and which are private
* If you deploy the Private gateway then all URL and private and only available within the VPC and subnet
* If you deploy in Public Gateway, then all URL will be publicly available and accessible from Internet
* When you deploy in cloud, the BASE URL will be generated dynamically.
* We will get this Base URL after completion fo deployment.

* Base URL Pattern: https://[Dynamic_ID].execute-api.[AWS_REGION].amazonaws.com
* Sample Base URL: https://8b3zmewgyl.execute-api.eu-central-1.amazonaws.com
* Here The list of Endpoints

| Name                       | Endpoint                                   | Query Parameters  |
|----------------------------|--------------------------------------------|-------------------|
| Facebook feed collector    | [base_url]/test/facebook_feed_collector    |                   |
| News feed collector        | [base_url]/test/news_feed_collector        |                   |
| Share price feed collector | [base_url]/test/share_price_feed_collector |                   |
| Twitter feed collector     | [base_url]/test/twitter_feed_collector     |                   |

### general Information

* This project is Terraform module-based
* File `terraform/module_all.tf` holds all `environment` modules
* After Deploying `api-datacollector`, some data will be collect and store in DynamoDB
* Jar file will store in AWS S3 by GitLab CI/CD pipeline on `master` branch commit
* No need to build the Java code locally
* Here API Gateway integration is private
    * You can access only from the same VPC and subnet
* When you run `deploy_infra.sh` the following files auto-created on your local computer
    * `.terraform`
    * `.terraform.lock.hcl`
    * `.terraform.tfstate.lock.info`
    * `plan.out`
    * `terraform.tfstate`
    * `terraform.tfstate.backup`
    * Those files keep the state of your deployment
    * Do not delete those file.
