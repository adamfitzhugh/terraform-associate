# Manage Infrastructure

## Providers
- Behaviours of resources rely on associated resource types known as providers.

- Each provider offers a set of named resource types and defines which arguments each resource accepts.

- Resource types belonging to the same provider share the same configuration, avoiding the need to repeat this information across every resource declaration.

### Provider Configuration
Example provider configuration:

```
provider "google" {
  project = "acme-app"
  region  = "us-central1"
}
```

- Terraform associates each resource type with a provider by taking the first word of the resource type name, so "google" is assumed to be the provider of the resource google_compute_instance.

- The body of the block defines arguments for the provider itself.

- Provider has two meta-arguments that are defined by Terraform and available for all provider blocks - version & alias.

- Everytime a provider is added to configuration, or a resource is added from that provider, Terraform must initialise it before it can be used. Initialisation downloads and installs the providers plugin so it can be executed later.

- Providers downloaded by 'terraform init' are only downloaded for the current working directory.

- When 'terraform init' is re-run with providers already installed, it will use an already-installed provider that meets constraints rather than download a new version. To upgrade, run a 'terraform init -upgrade'


## Third Party Plugins
- Third party providers must be manually installed since 'terraform init' cannot download them. These can be manually installed in the user plugins directory (~/.terraform.d/plugins). 'terraform init' can then initialise normally.

### Plugin Names & Versions

- The naming scheme for provider plugins is 'terraform-provider-<NAME>_vX.Y.Z' and Terraform uses the name to understand the version of a particular binary


## Terraform State

### Mapping to the Real World
- Terraform needs some sort of database to map Terraform config to the real world. For example, Terraform would map "aws_instance" "foo" as i-abcd1234.

### Metadata
- Alongside mapping resources and remote objects, Terraform must also track metadata such as resource dependencies.

- To ensure Terraform knows how to delete a resource, it retains a copy of the most recent set of dependencies within the state. Terraform can still determine the correct order for destruction from the state when you delete one or more items from the config file.

- One way to avoid using state is Terraform knowing a required ordering of operation between resources, however the complexity of this quickly increases. Terraform stores other metadata for similar reasons such as a pointer to the provider configuration that was most recently used with the resource in situations where multiple aliased providers are present.

### Performance
- To improve performance, Terraform stores a cached version of attribute values for all resources in the state.

- For small changes, running a 'terraform plan' or 'terraform apply' allows Terraform to query your providers and sync the latest attributes from all your resources. For larger changes querying every resource is too slow.

- To get around this, larger users of Terraform make use of the '-refresh=false' flag as well as the '-target' flag. In this instance, the cached state is treated as the record of truth.

### Syncing
- For teams using Terraform, using remote state is the preferred option. Terraform can use remote locking to avoid two people running Terraform at once and to make sure each Terraform run beings with the most up-to-date state.


## Terraform Settings

### Terraform Block Syntax