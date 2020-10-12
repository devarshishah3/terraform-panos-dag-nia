# Palo Alto Networks PAN-OS Dynamic Address Group (DAG) Tags module for Network Infrastructure Automation (NIA)

This Terraform module allows users to support **Dynamic Firewalling** by integrating [Consul](https://www.consul.io/) with Palo Alto Networks **PAN-OS** based [PA-Series and VM-Series NGFW](https://www.paloaltonetworks.com/network-security/next-generation-firewall) devices to dynamically manage dynamic registration/de-registration of **Dynamic Address Group (DAG) tags** based on services in Consul catalog.  

Using this Terraform module in conjunction with **consul-terraform-sync** enables teams to reduce manual ticketing processes and automate Day-2 operations related to application scale up/down in a way that is both declarative and repeatable across the organization and across multiple **PAN-OS** devices.

#### Note: This Terraform module is designed to be used only with **consul-terraform-sync**


## Feature
This module supports the following:
* Create, update and delete Dynamic Address Tags based service name and IP address for the service in Consul catalog. If service address is not defined in Consul catalog, node address is used instead.

If there is a missing feature or a bug - - [open an issue (to be updated) ](https://github.com/devarshishah3/terraform-panos-dag-nia/issues/new)

## What is consul-terraform-sync?

The **consul-terraform-sync** runs as a daemon that enables a **publisher-subscriber** paradigm between **Consul** and **PAN-OS** based devices to support **Network Infrastructure Automation (NIA)**. 

<p align="left">
<img width="800" src="https://user-images.githubusercontent.com/11891727/95024708-b5ec5900-0639-11eb-9fa5-c11a290a5305.png"> </a>
</p>

* consul-terraform-sync **subscribes to updates from the Consul catalog** and executes one or more automation **task** with appropriate value of *service variables* based on those updates. **consul-terraform-sync** leverages [Terraform](https://www.terraform.io/) as the underlying automation tool and utilizes the Terraform provider ecosystem to drive relevant change to the network infrastructure. 

* Each task consists of a runbook automation written as a compatible **Terraform module** using resources and data sources for the underlying network infrastructure provider.

Please refer to this [link](https://www.consul.io/docs/nia/installation/install) for getting started with **consul-terraform-sync**

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.13 |
| consul-terraform-sync | >= 0.1.0 |
| consul | >= 1.7 |

## Providers

| Name | Version |
|------|---------|
| panos | >= 1.6 |

## Compatibility
This module is meant for use with **consul-terraform-sync >= 0.1.0** and **Terraform >= 0.13** and **PAN-OS versions >= 8.0**

## Permissions
* In order for the module to work as expected, the user or the api_key associated to the **panos** Terraform provider must have **User-ID Agent** permissions enabled 


## Caveats
* PAN-OS versions >=9.0 have a behavior where the dynamic tags added to the address group will be present, but do not show up in the UI until the address group is associated to a policy. 


## Usage
In order to use this module, you will need to install **consul-terraform-sync**, create a **"task"** with this Terraform module as a source within the task, and run **consul-terraform-sync**.

The users can subscribe to the services in the consul catalog and define the Terraform module which will be executed when there are any updates to the subscribed services using a **"task"**.

**~> Note:** It is recommended to have the (consul-terraform-sync config guide)[https://www.consul.io/docs/nia/installation/configuration] for reference.  
1. Download the **consul-terraform-sync** on a node which is highly available (prefrably, a node running a consul client)
2. Add **consul-terraform-sync** to the PATH on that node
3. Check the installation
   ```
    $ consul-terraform-sync --version
   0.1.0
   Compatible with Terraform ~>0.13.0
   ```
 4. Create a config file **"tasks.hcl"** for consul-terraform-sync. Please note that this just an example. 
```terraform
log_level = <log_level> # eg. "info"

driver "terraform" {
  log = true
  required_providers {
    panos = {
      source = "PaloAltoNetworks/panos"
    }
  }
}

consul {
  address = "<consul agent address>" # eg. "1.1.1.1:8500"
}

provider "panos" {
  alias = "panos1" 
  hostname = "<panos_address>" # eg. "2.2.2.2"
  api_key  = "<api_key>" 
}



task {
  name = <name of the task (has to be unique)> # eg. "Create_DAG_on_PANOS1"
  description = <description of the task> # eg. "Dynamic Address Groups based on service definition"
  source = "devarshishah3/dag-nia/panos" # to be updated
  providers = ["panos.panos1"]
  services = ["<list of services you want to subscribe to>"] # eg. ["web", "api"]
  variable_files = ["<list of files that have user variables for this module (please input full path)>"] # eg. ["/opt/panw-config/user-demo.tfvars"]
}
```
 5. Start consul-terraform-sync
```
$ consul-terraform-sync -config-file=tasks.hcl
```
**consul-terraform-sync** will create right set of address groups and dynamic tags on the PAN-OS device based on the values in consul catalog.

**consul-terraform-sync is now subscribed to the Consul catalog. Any updates to the serices identified in the task will result in updating the address and the dyanmic address group tags on the PAN-OS devices** 



## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| vsys\_name | PAN-OS virual system name, e.g., vsys1 | `string` | vsys1 | yes |
| dag\_prefix | Prefix to be added to the dynamic address group on PAN-OS device created by consul-terraform-sync | `string` |  | no |
| dag\_prefix | Suffix to be added to the dynamic address group on PAN-OS device created by consul-terraform-sync | `string` |  | no |
| services | Consul services monitored by consul-terraform-sync | <pre>map(<br>    object({<br>      id        = string<br>      name      = string<br>      address   = string<br>      port      = number<br>      meta      = map(string)<br>      tags      = list(string)<br>      namespace = string<br>      status    = string<br><br>      node                  = string<br>      node_id               = string<br>      node_address          = string<br>      node_datacenter       = string<br>      node_tagged_addresses = map(string)<br>      node_meta             = map(string)<br>    })<br>  )</pre> | n/a | yes |


## Outputs

| Name | Description |
|------|-------------|
| address\_groups | Name of address groups dynamically created on PAN-OS device through consul-terraform-sync |
| tag_to_ip_associatuion | Name of the dynamic address tags created and the IP addresses associated |


  
