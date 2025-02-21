# K8ssandra AWS Terraform Module

## What is Elastic Kubernetes Service(EKS)?
[Amazon Elastic Kubernetes Service](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html) is a managed Kubernetes service on AWS, it gives user flexibility to run and scale Kubernetes applications in the AWS cloud.


## Terraform Resources created
* EKS cluster
* EKS node group
* IAM roles
* IAM role policy
* IAM role policy attachment
* S3 Bucket
* S3 Bucket public access block
* Virtual Private Cloud
* Subnets
* Security Groups
* Security Group rule
* NAT Gateway
* Internet Gateway
* elastic IP
* Route Table
* Route Table association

## Project directory Structure
<pre>
aws/
 ├──modules/
 |  ├──<a href="modules/s3/README.md">s3</a>
 |     ├── main.tf 
 |     └── variables.tf 
 |     └── outputs.tf 
 |     └── README.md 
 |  ├──<a href="modules/vpc/README.md">vpc</a>
 |     ├── main.tf 
 |     └── variables.tf 
 |     └── outputs.tf 
 |     └── README.md 
 |  ├──<a href="modules/iam/README.md">iam</a>
 |     ├── main.tf 
 |     └── variables.tf 
 |     └── outputs.tf 
 |     └── README.md
 |  ├──<a href="modules/eks/README.md">eks</a>
 |     ├── main.tf 
 |     └── variables.tf 
 |     └── outputs.tf 
 |     └── README.md
 |
 ├──<a href="env/README.md">env</a>
 |   ├── dev.tf
 |    ../modules/vpc
 |    ../modules/iam
 |    ../modules/eks
 |    ../modules/s3
 |  ├── version.tf 
 |  └── variables.tf 
 |  └── outputs.tf
 |  └── README.md
 |
 ├──<a href="scripts/README.md">scripts</a>
 |  ├── apply.sh
 |  └── common.sh
 |  └── delete_bucket.py
 |  └── destroy.sh
 |  └── init.sh
 |  └── make_bucket.py
 |  └── plan.sh
 |  └── README.md
 └──README.md
</pre>

## Prerequisites

|       NAME          |   Version  | 
|---------------------|------------|
| Terraform version   |   0.14     |
| aws provider        |   ~>3.0    |
| Helm version        |   v3.5.3   |
|   AWS CLI           |  version2  |
|   boto 3            |            |   
|  kubectl            |  1.17.17   |
|  python             |    3       |
|aws-iam-authenticator|   0.5.2    |

### Backend
  * Terraform uses persistent state data to keep track of the resources it manages. Since it needs the state in order to know which real-world infrastructure objects correspond to the resources in a configuration, everyone working with a given collection of infrastructure resources must be able to access the same state data.
  * Terraform backend configuration: 
  [Configuring your backend in aws s3](https://www.terraform.io/docs/language/settings/backends/s3.html)
  * Terraform state
  [How Terraform state works](https://www.terraform.io/docs/language/state/index.html)

Following Backend template is to store your backend remotely in s3 bucket, if you are planning to use remote backend copy the following sample template to [./env](#./env) folder and update the backend file with your bucket name, key and your bucket region.

If you don't want to use the remote backend, you can use the local local directory to store the state files. `terraform init` will generate a state file and it will be stored in your local directory under [./env/.terraform](./env).

Sample template to configure your backend in s3 bucket:
```
  terraform {
    backend "s3" {
      bucket = "<REPLACEME_bucket_name>"
      key    = "<REPLACEME_bucket_key>"
      region = "<REPLACEME_region>"
    }
  }
```

### Access

* Access to an existing AWS cloud as a owner or a developer. following permissions are required 
  * Managed policies( These policies are Managed by the AWS, you can locate them in the attach existing policies section).
    * AmazonEKSClusterPolicy
    * AmazonEKSWorkerNodePolicy
    * AmazonEKSServicePolicy
    * AmazonEKSVPCResourceController
  * Custom policy document located here.[policy_document](#./scripts/policy_document.json). In this JSON file there are three different policy documents. 
    * [IAM-Developer](#./scripts/policy_document.json/policy-for-IAM) Policy to manage IAM access to the user.
    * [AutoScaling Group](#./scripts/policy_document.json/Policy-for-AutoScaling-Group) Policy to manage AutoScaling Group access permissions.
    * [EC2&S3_policy](#./scripts/policy_document.json/policy-for-S3-and-EC2) Policy to manage EC2 and S3 permissions.
    you will need to create three different policies, we cannot able to add all the policies in the single document, it will exceeds the resource limit on the policy document.

### Tools

* Bash and common command line tools (Make, etc.)
* [Terraform v0.14.0+](https://www.terraform.io/downloads.html)
* [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
* [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) that matches the latest generally-available EKS cluster version.

#### Install Terraform

Terraform is used to automate the manipulation of cloud infrastructure. Its [Terraform installation instructions](https://www.terraform.io/intro/getting-started/install.html) are also available online.

#### AWS IAM authenticator

Amazon EKS uses IAM to provide authentication to your Kubernetes cluster through the AWS IAM authenticator for Kubernetes. Follow the instructions to install [aws-iam-authenticator installation instructions](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

#### Install kubectl
Kubernetes uses a command-line utility called `kubectl` for communicating with the cluster API server. The kubectl binary is available in many operating system package managers, and this option is often much easier than a manual download and install process. Follow the instructions to install [kubectl installation instructions](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html).

### Configure AWSCLI
Run `aws configure` command to configure your AWS cli, This Terraform module utilizes AWS CLI v2, if you have older versions of AWS CLI use the following instructions to uninstall older versions.

* [Uninstall AWS cli v1](https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html)
* [Install AWS cli v2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

```console
aws configure
```
Enter `access key`, `secret access key` default `region`.

## Test this project locally

Export the following terraform environment variables(TFVARS) for terraform to create the resources. if you don't want to export any values, you skip this part and proceed further. 

```console

# Environment name, eg. "dev"
# bash, zsh
export TF_VAR_environment=dev

#fish
set -x TF_VAR_environment dev

# Kubernetes cluster name, eg. "k8ssandra"
# bash, zsh
export TF_VAR_name=k8ssandra

#fish
set -x TF_VAR_name k8ssandra

# Resource Owner name, eg. "k8ssandra"
# bash, zsh
export TF_VAR_resource_owner=k8ssandra

#fish
set -x TF_VAR_resource_owner k8ssandra


# AWS region name, eg. "us-east-1" 
# bash, zsh
export TF_VAR_region=us-east-1

#fish
set -x TF_VAR_region us-east-1

```

Important: Initialize the terraform modules and downloads required providers.

```console
cd env/

terraform init
```

Run the following commands to plan and apply changes to your infrastructure.

If you miss export any values or don't want to import any values, by running the following commands they are going prompt for required values in the `variable.tf`. you can enter those values through command-line.

```console 
terraform plan

terraform apply
```

To destroy the resource, use the following instructions:

It is important to export or pass the right values when destroying the resources on a local workspace. Make sure you exported(TF_VAR) or enter right environment variables.

Verify the resources before you destroy Used the following command.

```console
terraform plan -destroy
```

Run the following command to destroy all the resources in your local workspace.

```console
terraform destroy
```
or 

```console
terraform destroy -auto-approve
```
