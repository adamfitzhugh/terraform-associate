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

- The naming scheme for provider plugins is 'terraform-provider-<NAME>_vX.Y.Z' and Terraform uses the name to understand the version of a particular binary.

## Terraform State

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

### Configuring a Terraform Backend
- The Terraform backend defines where and how operations are performed, where state is stored etc.

### Terraform Required Versions
- You can use the 'required_version' setting to specify which versions of Terraform CLI can be used. If the version doesnt match, Terraform will produce an error.

- Constraint operations are allowed to specify a version:

```
= (or no operator): exact version

!=: version not equal

>, >=, <, <=: version comparison

~>: pessimistic constraint operator, constraining both oldest and newest version allowed. For example '~> 0.9' is equal to '>= 0.9', '< 1.0'
```

- Re-usable modules should constrain only the minimum allowed version, such as '>= 1.0.0'

### Experimental Language Features
- From time to time Terraform will introduce new features initially via an opt-in experiment so the community can try the new feature and give feedback.

- In releases where they are available, you can allow them on a per module basis by using the 'experiments' parameter, for example:

```
terraform {
  experiments = [example]
}
```

- Hashicorp do not recommend using experimental features in modules for production use.

- When using experimental features a warning will appear on every 'terraform plan' & 'terraform apply'


## Provisioners
- Provisioners can be used to run specific actions on the local/remote machine in order to prepare servers or other infrastructure for production.

### Provisioners are a Last Resort
- Terraform only includes provisioners knowing there will always be certain behaviours that can't be represented in Terraform's declarative model.

- This adds complexity. Terraform cannot model actions of provisioners in a plan/apply as it could in effect be any action. Also with successful provisioners, Terraform would need to understand direct network access to servers, and know the login credentials for the devices.

### Passing data to Virtual Machines
- Terraform recommends passing this data in as metadata or using a defined cloud platform mechanism to pass in the data such as a start up script as part of the module

- If the feature is unavailable, as a short-term offering you should use the local-exec provisioner.

### The 'self' Object
- Expressions in provisioner blocks cannot refer to the parent resource by name, they must use the 'self' object. For example, use 'self.public_ip' to reference an aws_instance IP.

```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo The server's IP address is ${self.private_ip}"
  }
}
```

### Creation-Time Provisioners
- By default provisioners only run when the resource is created and not during an update or any other lifecycle.

- If a creation-time provisioner fails, that resource is marked as tainted and will be planned for destruction and recreation upon the next 'terraform apply'.

### Destroy-Time Provisioners
- If 'when = destroy' is specified, the provisioner will run when the resource is being destroyed.

```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = "destroy"
    command = "echo 'Destroy-time provisioner'"
  }
}
```

- If they fail, Terraform will error and rerunt he provisioners again on the next 'terraform apply'.

### Multiple Provisioners
- You can specify multiple provisioners within a resource. These are ran in the order they're defined in the configuration file.

### Failure Behaviour
- By default if a provisioner fails then Terraform will fail. This can be overridden by using the 'on_failure' setting. The allowed values are 'continue' (ignore the error and continue with the creation or destruction) and 'fail' (raise an error and stop the apply)