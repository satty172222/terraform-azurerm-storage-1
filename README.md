# Azure Storage Account Terraform Module

Terraform Module to create an Azure storage account with a set of containers (and access level), set of file shares (and quota), tables, queues, Network policies and Blob lifecycle management.

To defines the kind of account, set the argument to `account_kind = "StorageV2"`. Account kind defaults to `StorageV2`. If you want to change this value to other storage accounts kind, then this module automatically computes the appropriate values for `account_tier`, `account_replication_type`. The valid options are `BlobStorage`, `BlockBlobStorage`, `FileStorage`, `Storage` and `StorageV2`.

>Note: *static_website can only be set when the account_kind is set to `StorageV2`.*

These types of resources are supported:

* [Storage Account](https://www.terraform.io/docs/providers/azurerm/r/storage_account.html)
* [Storage Advanced Threat Protection](https://www.terraform.io/docs/providers/azurerm/r/advanced_threat_protection.html)
* [Containers](https://www.terraform.io/docs/providers/azurerm/r/storage_container.html)
* [SMB File Shares](https://www.terraform.io/docs/providers/azurerm/r/storage_share.html)
* [Storage Table](https://www.terraform.io/docs/providers/azurerm/r/storage_table.html)
* [Storage Queue](https://www.terraform.io/docs/providers/azurerm/r/storage_queue.html)
* [Network Policies](https://www.terraform.io/docs/providers/azurerm/r/storage_account.html#network_rules)
* [Azure Blob storage lifecycle](https://www.terraform.io/docs/providers/azurerm/r/storage_management_policy.html)

## Module Usage

```hcl
module "storage" {
  source  = "kumarvna/storage/azurerm"
  version = "2.1.0"

  # By default, this module will not create a resource group
  # proivde a name to use an existing resource group, specify the existing resource group name,
  # and set the argument to `create_resource_group = false`. Location will be same as existing RG.
  resource_group_name  = "rg-demo-internal-shared-westeurope-002"
  location             = "westeurope"
  storage_account_name = "mydefaultstorage"

  # To enable advanced threat protection set argument to `true`
  enable_advanced_threat_protection = true

  # Container lists with access_type to create
  containers_list = [
    { name = "mystore250", access_type = "private" },
    { name = "blobstore251", access_type = "blob" },
    { name = "containter252", access_type = "container" }
  ]

  # SMB file share with quota (GB) to create
  file_shares = [
    { name = "smbfileshare1", quota = 50 },
    { name = "smbfileshare2", quota = 50 }
  ]

  # Storage tables
  tables = ["table1", "table2", "table3"]

  # Storage queues
  queues = ["queue1", "queue2"]

  # Adding TAG's to your Azure resources (Required)
  # ProjectName and Env are already declared above, to use them here, create a varible.
  tags = {
    ProjectName  = "demo-internal"
    Env          = "dev"
    Owner        = "user@example.com"
    BusinessUnit = "CORP"
    ServiceClass = "Gold"
  }
}
```

## Create resource group

By default, this module will not create a resource group and the name of an existing resource group to be given in an argument `resource_group_name`. If you want to create a new resource group, set the argument `create_resource_group = true`.

>*If you are using an existing resource group, then this module uses the same resource group location to create all resources in this module.*

## BlockBlobStorage accounts

A BlockBlobStorage account is a specialized storage account in the premium performance tier for storing unstructured object data as block blobs or append blobs. Compared with general-purpose v2 and BlobStorage accounts, BlockBlobStorage accounts provide low, consistent latency and higher transaction rates.

BlockBlobStorage accounts don't currently support tiering to hot, cool, or archive access tiers. This type of storage account does not support page blobs, tables, or queues.

To create BlockBlobStorage accounts, set the argument to `account_kind = "BlockBlobStorage"`.

## FileStorage accounts

A FileStorage account is a specialized storage account used to store and create premium file shares. This storage account kind supports files but not block blobs, append blobs, page blobs, tables, or queues.

FileStorage accounts offer unique performance dedicated characteristics such as IOPS bursting. For more information on these characteristics, see the File share storage tiers section of the Files planning guide.

To create BlockBlobStorage accounts, set the argument to `account_kind = "FileStorage"`.

## Containers

A container organizes a set of blobs, similar to a directory in a file system. A storage account can include an unlimited number of containers, and a container can store an unlimited number of blobs. The container name must be lowercase.

This module creates the containers based on your input within an Azure Storage Account.  Configure the `access_type` for this Container as per your preference. Possible values are `blob`, `container` or `private`. Preferred Defaults to `private`.

## SMB File Shares

Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block (SMB) protocol. Azure file shares can be mounted concurrently by cloud or on-premises deployments of Windows, Linux, and macOS.

This module creates the SMB file shares based on your input within an Azure Storage Account.  Configure the `quota` for this file share as per your preference. The maximum size of the share, in gigabytes. For Standard storage accounts, this must be greater than `0` and less than `5120` GB (5 TB). For Premium FileStorage storage accounts, this must be greater than `100` GB and less than `102400` GB (100 TB).

## Soft delete for Blob storage

Soft delete protects blob data from being accidentally or erroneously modified or deleted. When soft delete is enabled for a storage account, blobs, blob versions (preview), and snapshots in that storage account may be recovered after they are deleted, within a retention period that you specify.

This module allows you to specify the number of days that the blob should be retained period using `soft_delete_retention` argument between 1 and 365 days.

## Configure Azure Storage firewalls and virtual networks

The Azure storage firewall provides access control access for the public endpoints of the storage account.  Use network policies to block all access through the public endpoint when using private endpoints. The storage firewall configuration also enables select trusted Azure platform services to access the storage account securely.

The default action set to `Allow` when no network rules matched. A `subnet_ids` or `ip_rules` can be added to `network_rules` block to allow a request that is not Azure Services.

```hcl
module "storage" {
  source  = "kumarvna/storage/azurerm"
  version = "2.1.0"

  # .... omitted

  # If specifying network_rules, one of either `ip_rules` or `subnet_ids` must be specified
  network_rules = {
    bypass     = ["AzureServices"]
    # One or more IP Addresses, or CIDR Blocks to access this Key Vault.
    ip_rules   = ["123.201.18.148"]
    # One or more Subnet ID's to access this Key Vault.
    subnet_ids = []
  }

  # .... omitted
  }
  ```

## Manage the Azure Blob storage lifecycle

Azure Blob storage lifecycle management offers a rich, rule-based policy for General Purpose v2 (GPv2) accounts, Blob storage accounts, and Premium Block Blob storage accounts. Use the policy to transition your data to the appropriate access tiers or expire at the end of the data's lifecycle.

The lifecycle management policy lets you:

* Transition blobs to a cooler storage tier (hot to cool, hot to archive, or cool to archive) to optimize for performance and cost
* Delete blobs at the end of their lifecycles
* Define rules to be run once per day at the storage account level
* Apply rules to containers or a subset of blobs*

This module supports the implementation of storage lifecycle management. If specifying network_rules, one of either `ip_rules` or `subnet_ids` must be specified and default_action must be set to `Deny`.

```hcl
module "storage" {
  source  = "kumarvna/storage/azurerm"
  version = "2.1.0"

  # .... omitted

  # Lifecycle management for storage account.
  # Must specify the value to each argument and default is `0`
  lifecycles = [
    {
      prefix_match               = ["mystore250/folder_path"]
      tier_to_cool_after_days    = 0
      tier_to_archive_after_days = 50
      delete_after_days          = 100
      snapshot_delete_after_days = 30
    },
    {
      prefix_match               = ["blobstore251/another_path"]
      tier_to_cool_after_days    = 0
      tier_to_archive_after_days = 30
      delete_after_days          = 75
      snapshot_delete_after_days = 30
    }
  ]

  # .... omitted
  }
  ```

## Recommended naming and tagging conventions

Well-defined naming and metadata tagging conventions help to quickly locate and manage resources. These conventions also help associate cloud usage costs with business teams via chargeback and show back accounting mechanisms.

> ### Resource naming

An effective naming convention assembles resource names by using important resource information as parts of a resource's name. For example, using these [recommended naming conventions](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#example-names), a public IP resource for a production SharePoint workload is named like this: `pip-sharepoint-prod-westus-001`.

> ### Metadata tags

When applying metadata tags to the cloud resources, you can include information about those assets that couldn't be included in the resource name. You can use that information to perform more sophisticated filtering and reporting on resources. This information can be used by IT or business teams to find resources or generate reports about resource usage and billing.

The following list provides the recommended common tags that capture important context and information about resources. Use this list as a starting point to establish your tagging conventions.

Tag Name|Description|Key|Example Value|Required?
--------|-----------|---|-------------|---------|
Project Name|Name of the Project for the infra is created. This is mandatory to create a resource names.|ProjectName|{Project name}|Yes
Application Name|Name of the application, service, or workload the resource is associated with.|ApplicationName|{app name}|Yes
Approver|Name Person responsible for approving costs related to this resource.|Approver|{email}|Yes
Business Unit|Top-level division of your company that owns the subscription or workload the resource belongs to. In smaller organizations, this may represent a single corporate or shared top-level organizational element.|BusinessUnit|FINANCE, MARKETING,{Product Name},CORP,SHARED|Yes
Cost Center|Accounting cost center associated with this resource.|CostCenter|{number}|Yes
Disaster Recovery|Business criticality of this application, workload, or service.|DR|Mission Critical, Critical, Essential|Yes
Environment|Deployment environment of this application, workload, or service.|Env|Prod, Dev, QA, Stage, Test|Yes
Owner Name|Owner of the application, workload, or service.|Owner|{email}|Yes
Requester Name|User that requested the creation of this application.|Requestor| {email}|Yes
Service Class|Service Level Agreement level of this application, workload, or service.|ServiceClass|Dev, Bronze, Silver, Gold|Yes
Start Date of the project|Date when this application, workload, or service was first deployed.|StartDate|{date}|No
End Date of the Project|Date when this application, workload, or service is planned to be retired.|EndDate|{date}|No

> This module allows you to manage the above metadata tags directly or as a variable using `variables.tf`. All Azure resources which support tagging can be tagged by specifying key-values in argument `tags`. Tag `ResourceName` is added automatically to all resources.

```hcl
module "key-vault" {
  source  = "kumarvna/storage/azurerm"
  version = "2.1.0"

  # ... omitted

  tags = {
    ProjectName  = "demo-project"
    Env          = "dev"
    Owner        = "user@example.com"
    BusinessUnit = "CORP"
    ServiceClass = "Gold"
  }
}  
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.13 |
| azurerm | ~> 2.27 |

## Providers

| Name | Version |
|------|---------|
| azurerm | ~> 2.27 |
| null | n/a |
| random | n/a |

## Inputs

Name | Description | Type | Default
---- | ----------- | ---- | -------
`create_resource_group`|Whether to create resource group and use it for all networking resources|string| `false`
`resource_group_name`|The name of the resource group in which resources are created|string|`""`
`location`|The location of the resource group in which resources are created|string| `""`
`account_kind`|General-purpose v2 accounts: Basic storage account type for blobs, files, queues, and tables.|string|`"StorageV2"`
`skuname`|The SKUs supported by Microsoft Azure Storage. Valid options are Premium_LRS, Premium_ZRS, Standard_GRS, Standard_GZRS, Standard_LRS, Standard_RAGRS, Standard_RAGZRS, Standard_ZRS|string|`Standard_RAGRS`
`access_tier`|Defines the access tier for BlobStorage and StorageV2 accounts. Valid options are Hot and Cool.|string|`"Hot"`
`assign_identity`|Set to `true` to enable system-assigned managed identity, or `false` to disable it.|string|`true`
`soft_delete_retention`|Number of retention days for soft delete. If set to null it will disable soft delete all together.|number|`30`
`enable_advanced_threat_protection`|Controls Advance threat protection plan for Storage account!string|`false`
`network_rules`|Configure Azure storage firewalls and virtual networks|list|`null`
`containers_list`| List of container|list|`[]`
`file_shares`|List of SMB file shares|list|`[]`
`queues`|List of storages queues|list|`[]`
`tables`|List of storage tables|list|`[]`
`lifecycles`|Configure Azure Storage firewalls and virtual networks|list|`{}`
`Tags`|A map of tags to add to all resources|map|`{}`

### `Container` objects (must have keys)

Name | Description | Type | Default
---- | ----------- | ---- | -------
`name` | Name of the container | string | `""`
`access_type` | The Access Level configured for the Container. Possible values are `blob`, `container` or `private`.|string|`"private"`

### `SMB file Shares` objects (must have keys)

Name | Description | Type | Default
---- | ----------- | ---- | -------
`name` | Name of the SMB file share | string | `""`
`quota` | The required size in GB. Defaults to `5120`|string|`""`

### `network_rules` objects (must have keys)

Name | Description | Type | Default
---- | ----------- | ---- | -------
`bypass`|Specifies whether traffic is bypassed for Logging/Metrics/AzureServices. Valid options are any combination of `Logging`, `Metrics`, `AzureServices`, or `None`.|string |`"AzureServices"`
`ip_rules`|List of public IP or IP ranges in CIDR Format. Only IPV4 addresses are allowed. Private IP address ranges are not allowed.|list(string)|`[]`
subnet_ids|A list of resource ids for subnets.|list(string)|`[]`

### `lifecycles` objects (must have keys)

Name | Description | Type | Default
---- | ----------- | ---- | -------
`prefix_match`|An array of strings for prefixes to be matched|set(string)|`[]`
`tier_to_cool_after_days`|The age in days after last modification to tier blobs to cool storage. Supports blob currently at `Hot` tier. Must be at least `0`.|number|`0`
`tier_to_archive_after_days`|The age in days after last modification to tier blobs to archive storage. Supports blob currently at `Hot` or `Cool` tier. Must be at least `0`.|number|`0`
`delete_after_days`|The age in days after last modification to delete the blob. Must be at least 0.|number|`0`
`snapshot_delete_after_days`|The age in days after create to delete the snapshot. Must be at least 0.|number|`0`

## Outputs

Name | Description
---- | -----------
`resource_group_name`|The name of the resource group in which resources are created
`resource_group_id`|The id of the resource group in which resources are created
`resource_group_location`|The location of the resource group in which resources are created
`storage_account_id`|The ID of the storage account
`sorage_account_name`|The name of the storage account
`storage_account_primary_location`|The primary location of the storage account
`storage_account_primary_web_endpoint`|The endpoint URL for web storage in the primary location
`storage_account_primary_web_host`|The hostname with port if applicable for web storage in the primary location
`storage_primary_connection_string`|The primary connection string for the storage account
`storage_primary_access_key`|The primary access key for the storage account
`storage_secondary_access_key`|The secondary access key for the storage account
`containers`|The list of containers
`file_shares`|The list of SMB file shares
`tables`|The list of storage tables
`queues`|The list of storage queues

## Resource Graph

![Resource Graph](graph.png)

## Authors

Originally created by [Kumaraswamy Vithanala](mailto:kumarvna@gmail.com)

## Other resources

* [Azure Storage documentation](https://docs.microsoft.com/en-us/azure/storage/)
* [Terraform AzureRM Provider Documentation](https://www.terraform.io/docs/providers/azurerm/index.html)
