# Google Compute Network
This is a Dynamic module in Terraform to create compute Network(VPC). This module will be called from the ./env/dev.tf file. 

* main.tf : contains all the resources, which will be created with `terraform apply` command.
* variables.tf : contains all the variables required to create the resources.
* outputs.tf : print output attributes of the resources.


## Google cloud resources created
* Google Compute Network(VPC)
* Compute Subnet
* Google Compute Firewall
* Compute Router
* Compute Router NAT


## Providers

|       NAME        |   Version  | 
|-------------------|------------|
| terraform version |   0.14     |
| GCP provider      |   ~>3.0    |

## Inputs

|       Name        |   Description  |  Type  |  Required    |
|-------------------|----------------|--------|:------------:|
| name |  name of the cluster and prefix of the related resources names | `string` | yes |
| project_id |  Id of the project which holds the components | `string` | yes |
| region | the region to create the vpc network | `string` | yes |
| cidr_block | The IP address range of the VPC in CIDR notation. | `string` | no |
| cidr_subnetwork_width_delta | The difference between your network and subnetwork netmask | `string` | no |
| cidr_subnetwork_spacing | How many subnetwork-mask sized spaces to leave between each subnetwork type | `string` | no |
| secondary_cidr_block | The IP address range of the VPC's secondary address range | `string` | no |
| secondary_cidr_subnetwork_width_delta | The difference between your network and subnetwork's | `string` | no |
| secondary_cidr_subnetwork_spacing | How many subnetwork-mask sized spaces to leave | `string` | no |
| enable_cloud_nat | Whether to enable Cloud NAT. | `bool` | no |
| enable_cloud_nat_logging | Whether the NAT should export logs. | `bool` | no |
| cloud_nat_logging_filter | What filtering should be applied to logs | `string` | no |



## Outputs

|    Name     |    description   | 
|-------------|:----------------:|
| network_selflink | compute network link |
| subnetwork_selflink | subnetwork compute link |
