# Read and Write Configuration

## Resources
- Resources describe one or more infrastructure objects.


### Resource Syntax
- A resource block declares a resource type with a given local name. This name can be used elsewhere in the Terraform code but has no significance outside of its module.

```
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

- The resource type and name must be unique within a module.


### Resource Behaviour
- When Terraform creates new infrastructure based on the resource block, the identifier for that object is saved within Terraform's state, allowing it to be managed by Terraform in future changes/updates.

- Meta-arguments within resource blocks allow details of the standard resource behaviour to be customised on a per-resource basis.

#### Resource Dependencies
- Some Terraform resources can be managed alone which allows Terraform to make multiple changes to resources in parallel. However, other resources depend on other specific resources in order to be updated, mainly because this is how that particular resource works. Most of these dependencies are handled automatically.

- Terraform is able to analyse expressions within a resource to find references to other objects and handle them accordingly.

- In some cases where Terraform is unable to recognise dependencies there is a 'depends_on' meta-argument which can explicitly specify a dependency.


### Meta-Arguments
- The following meta-arguments currently exist within Terraform:
    - 'depends_on' - for specifying hidden dependencies.
    - 'count' - for creating multiple resources based on the count.
    - 'for_each' - create multiple instances according to the map or set of strings.
    - 'provider' - for selecting non-default provider configuration.
    - 'lifecycle' - for lifecycle customisations.
    - provisioner and 'connection' - used for extra actions after resource creation.

#### depends_on
- Handles hidden resource dependencies that Terraform cannot automatically infer.

- This argument is available in all resource blocks regardless of resource type.

- If this argument is present, it must reference other resources in the same module.

#### count
- 'count' accepts whole numbers and creates that many instances for that particular resource.

```
resource "aws_instance" "server" {
  count = 4 # create four similar EC2 instances

  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${count.index}"
  }
}
```

#### for_each
- When your resources are identical, 'count' is appropriate. If some of their arguments need different values, it's safer to use 'for_each'.

- 'for_each' accepts a map or set of strings and creates an instance for each item in that map or set. Each instance has a distinct object associated with it and is separately created/updated/destroyed when Terraform is applied.

```
resource "azurerm_resource_group" "rg" {
  for_each = {
    a_group = "eastus"
    another_group = "westus2"
  }
  name     = each.key
  location = each.value
}
```

- When 'for_each' is set, Terraform distinguishes between the resource block itself and the multiple resource instances associated with it.

#### provider
- The 'provider' meta-argument overrides Terraform's default behaviour of selecting a provider configuration based on the resource type name.

- By default, Terraform would associate 'google_compute_instance' with the provider named 'google'.

- By using the 'provider' meta-argument an aliased provider configuration can be selected:

```
# default configuration
provider "google" {
  region = "us-central1"
}

# alternative, aliased configuration
provider "google" {
  alias  = "europe"
  region = "europe-west1"
}

resource "google_compute_instance" "example" {
  # This "provider" meta-argument selects the google provider
  # configuration whose alias is "europe", rather than the
  # default configuration.
  provider = google.europe

  # ...
}
```

#### lifecycle
- Within lifecycle, the following meta-arguments are supported:
    - 'create_before_destroy' - by default when Terraform cannot make an in-place update to a resource it will destroy it and then create a new object. 'create_before_destroy' tells Terraform to create a new object before destroying the old object.
    - 'prevent_destroy' - this will cause Terraform to reject with any error any plan that would destroy the infrastructure this is configured under.
    - 'ignore_changes' - by default Terraform plans to update the remote object to match configuration on it's next run. By using 'ignore_changes' this amends the default behaviour by and tells Terraform to ignore updating specific attributes.

#### provisioner and connection
- Some infrastructure objects may require further actions before they become fully functional. For example, compute instances may require scripts to be ran before they can be used for their intended operation. These can be defined by resource provisioners.

- Provisioners though represent non-declarative actions taken during the creation of the resource.


### Operation Timeouts
- You are able to declare operation timeouts that allow you to customise how long certain operations should taken before being considered to have failed. For example, 'aws_db_instance' allows configurable timeouts for create, update and delete operations. Most resource types however do not support the timeout block.

```
resource "aws_db_instance" "example" {
  # ...

  timeouts {
    create = "60m"
    delete = "2h"
  }
}
```


## Data Sources
- Data sources allow data to be fetched for use elsewhere in Terraform configuration. Using data sources allows Terraform configuration to make use of information defined outside of Terraform.

### Using Data Sources

```
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

- This configuration requests that Terraform reads from a given data source 'aws_ami' and export the resul under the local name 'example'.

- Data sources are often obtained during a Terraform refresh which ensures the data retrieved is available for use.

- Within the block of code, the data instance will export one or more attributes which can be used in other resources as reference expressions.

### Data Source Arguments
- Each data resource is associated with a single data source which determines the kind of object it reads and what query constraint arguments are available.

- Each data source in turn belongs to a provider.

### Data Resource Behaviour
- Data is mainly retrieved on a Terraform refresh before being applied. However, there are instances when it cannot be determined until after configuration is applied. In this case reading from the data source is deferred until the apply phase.

### Local-only Data Sources
- Some data sources only operate locally, calculating results and exposing them elsewhere. For example, rendering templates, reading local files and rendering IAM policies.

### Data Resource Dependencies
- Data resources have the same dependency behaviour as defined in resources. For example, the 'depends_on' meta-argument has the same behaviour here.

- Using 'depends_on' with data resources isn't recommended as it is forced to be read at the apply phase.

### Multiple Resource Instances
- Data resources also support 'count' and 'for_each' meta-arguments with the same syntax behaviour.

- When using these it's important to distinguish the resource itself from the instances it creates. Each instance will separately read from its data source with it's own variants.

### Selecting a Non-default Provider Configuration
- The 'provider' meta-argument is also supported with the same syntax and behaviour.

### Lifecycle Customisations
- Data resources do not have any customisation settings available for their lifecycle.


## Resource Addressing
- A resource address is a string that references a specific resource in a larger infrastructure. An address is made up of 2 parts: module path & resource spec.

- Module Path - addresses a module within a tree of modules.
- Resource Spec - addresses a specific resource in the config.

- For example, given the Terraform config includes:

```
resource "aws_instance" "web" {
  # ...
  count = 4
}
```

- An address like this refers to the last instance in the config:

```
aws_instance.web[3]
```

- An address like this refers to all 4 instances:

```
aws_instance.web
```

## References to Named Values
- Terraform makes several kinds of names available. Each of these names you can use as single expressions, or combine then with other expressions to compute new values.

- The following are available:
    - <RESOURCE TYPE>.<NAME> - represents the managed resource and its name.
    - var.<NAME> - input variable of a given name.
    - local.<NAME> - local value of a given name.
    - module.<MODULE NAME>.<OUTPUT NAME> - value of an output value from a child module called from a current module.
    - data.<DATA TYPE>.<NAME> - represents a data resource of the given data source type and name.
    - path.module - filesystem of the module where the expression is located.
    - path.root - filesystem of the root module of the configuration.
    - path.cwd - path of the current working directory.
    - terraform.workspace - name of the currently selected workspace.

### Named Values and Dependencies
- Terraform analyses expressions in configuration to create dependencies between objects. For example, an expression in a resource argument that refers to another managed resource creates a dependency between those two resources.

### References to Resource Attributes
- Consider the following configuration. A number of attributes are able to be exported from this resource type:

```
resource "aws_instance" "example" {
  ami           = "ami-abc123"
  instance_type = "t2.micro"

  ebs_block_device {
    device_name = "sda2"
    volume_size = 16
  }
  ebs_block_device {
    device_name = "sda3"
    volume_size = 20
  }
}
```

- 'aws_instance.example.ami' can be used to extract the AMI name of this resource.
- 'aws_instance.example.id' can be used to extract the ID of this resource.
- 'aws_instance.example.ebs_block_device[*].device_name' will give a list of all the device_name values for EBS. This is known as a splat expression as it's taken from a nested block.

- Consider the below example:

```
    device "foo" {
      size = 2
    }
    device "bar" {
      size = 4
    }
```

- Sometimes nested blocks are defined as taking a logical key to identify each block which servers a similar purpose as the resources own name by providing a way to refer to that single block.
- Arguments inside blocks with keys can be accessed like the following: 'aws_instance.example.device["foo"].size'

- In order to access a single instance in a resource which has the 'count' argument set, you will need to use the index syntax, for example: 'aws_instance.example[0].id'

### Values not yet Known
- In some cases, Terraform does not know the value of some resource values until it has been created. In this case, Terraform uses a special "unknown value" for these. The Terraform language automatically handles these during expressions.

- Unknown values appear in the 'terraform plan' as "(not yet known)".


## Type Constraints

### Complex Types
- A complex type is a type that groups multiple values into a single value. There are two complex types: collection types (which group similar values) and structural types (which group dissimilar values).

#### Collection Types
- Allows multiple values of ONE type to be grouped together as a single value. The type of value within a collection is known as an 'element type' and all collection types must have one.

- For example 'list(string)' indicates a list of strings and 'list(number)' indicates a list of numbers - all elements must be of the same type.

- Three kinds of collection type in Terraform include:
    - 'list(...)' - sequence of values identified by whole numbers starting with zero. List accepts any element type as long as they are all the same.
    - 'map(...)' - collection of values where each is identified by a string label. Map accepts any element type as long as they are all the same.
    - 'set(...)' - collection of values that do not have any secondary identifiers or ordering.

#### Structural Types
- Allows multiple values of several types to be grouped together as a single value. For example:
    - 'object(...)' - a collection of named attributes that each have their own type. The schema for object types is { <KEY> = <TYPE>, <KEY> = <TYPE>, ... }. All key/value pairs must be specified under the object type.
    - 'tuple(...)' - a sequence of elements identified by consecutive whole numbers starting with zero. The schema for tuples is [<TYPE>, <TYPE>, ...]. Values that match the tuple type must have exactly the same number of elements.

- For example, an object type of object{{ name=string, age=number}} would match the below:

```
{
  name = "John"
  age  = 52
}
```

- A tuple type of tuple([string, number, bool]) would match the below:

```
["a", 15, true]
```

#### Conversion of Complex Types
- Terraform has the ability to convert similar complex types. For example a map can be converted to an object provided it has at least the keys required by the object schema. A list can also be converted to a tuple only if it has the exact required number of elements, and sets are almost similar to tuples and lists.

### dynamic Blocks
- Within top level block constructs such as resources, expressions can usually be used only when assigning a value to an argument using the 'name = expression' format.

- You can also dynamically construct repeatable nested blocks like 'setting' using a special 'dynamic' block type which is supported in 'resource', 'data', 'provider' and 'provisioner' blocks.

```
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name                = "tf-test-name"
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}
```

- 'dynamic' blocks act as a for loop but produce nested blocks instead of a complex typed value. It will iterate over a given value and generates a nested block for each element of that value.

- A dynamic blockc an only generate arguments that belong to a resource type, data source, provider or provisioner being configured.


### Vault
- Run Terraform code with a Vault token which in turn looks in Vault for specific secrets to configure those resources.

- Consul is a database in this instance used for storing Vault secrets. These secrets can be backed up using Consul.

- Example lab - installing 3 Consul nodes, 3 Vault nodes and a bastion host split across AZ's.

- Vault comes up in a sealed state by default and doesn't process traffic.