# Modules

- Writing all your Terraform configurations in a single file or directory can cause many problems:
    - Understanding the configuration file itself can become difficult.
    - Updating the configuration will become more risky.
    - There will become an increasing amount of duplication across dev/staging/production environments.
    - Sharing configurations between projects and teams can become more difficult


### What are Modules used for?
Modules can help solve the problems listed above:

- Organise Configuration - Modules make it easier to navigate, understand and update your configuration by keeping relevant parts together.
- Encapsulate Configuration - Encapsulating configuration helps prevent unintended consequences, such as a change to one part of your configuration accidentally causes changes to other infrastructure.
- Re-use Configuration - Using modules can be less error-prone and save time by re-using configuration. 
- Provides Consistency - Modules help provide consistency. It helps ensure best-practices are applied across all your configuration.


### What is a Terraform Module
- A Terraform module contains a set of Terraform configuration files in a single directory.

- When you run Terraform commands directly from a directory it's considered a 'root module'. 

An example module:
```
$ tree minimal-module/
.
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

- Terraform installs newly installed modules in the .terraform/modules directory when a 'terraform init' or 'terraform get' are ran. For local modules, Terraform will create a symlink to the modules directory.

- Terraform will inherit the 'provider' module from the enclosing configuration. It is not advised to add the provider blocks to modules.

- Whenever you add a new module to the configuration it must be installed by Terraform before being used. This can be done by either 'terraform get' or a 'terraform init'.

#### Calling Modules
- When running Terraform from a directory, that code may call modules in other directories. When this happens, Terraform loads and processes that modules configuration files. A module that is called by another configuration is sometimes referred to as a 'child module'.

#### Calling a Child Module
- A module that includes a module block is the calling module of the child module.

- All modules require the 'source' argument and its value should be a literal string.

- After adding, removing or modifying module blocks you must re-run 'terraform init' to allow Terraform to adjust the installed modules.

#### Local & Remote Modules
- Modules can either be loaded from a local filesystem or remote source.


### Modules Best Practices
- Start writing your configuration with modules in mind. 

- Use local modules to organise and encapsulate your code. This will reduce the burden of maintaining and updated your configuration as your infrastructure grows and becomes more complex.

- Using the public Terraform registry to find useful modules. This allows you to more quickly and efficiently implement your configuration.

- Publish & share modules with the wider team. This allows for less error-prone and more consistent configurations across an organisation.


### Using Modules
- You are able to search modules on the Terraform Registry page. Only verified module results are shown however you are able to look for unverified modules also.

- The syntax for referencing a specific registry module is <NAMESPACE>/<NAME>/<PROVIDER>. For example: hashicorp/consul/aws.

- For some modules, there are required inputs you must set before being able to use the module. These are subsequently downloaded and cached when you run a 'terraform init'.

### Module Versions
- Terraform recommend explicitly adding a module version for each external module to avoid unexpected behaviour. You can specify the version attribute as shown below:

```
module "consul" {
  source  = "hashicorp/consul/aws"
  version = "0.0.5"

  servers = 3
}
```

- You can use expressions when specifying versions such as '<=', '>=', '~>'


### Input Variables

#### Declaring Input Variables
- The name of each variable must be unique among all variables in the same module.

- Variables can be named anything except the following: 'source', 'version', 'providers', 'count', 'for_each', 'lifecycle', 'depends_on', 'locals'

- The 'default' option in variables will be used if no other value is set. The 'default' argument requires a literal value and cannot reference other objects in the configuration.

#### Type Constraints
- The 'type' argument allows you to restrict the type of value that will be accepted for that variable.

- Supported type constraints are: 'string', 'number', 'bool'

- Supported type constructors are: 'list', 'set', 'map', 'object', 'tuple'

- If both the 'type' and 'default' arguments are given then that value must be convertible to the specified type.

#### Custom Validation Rules
- In addition to type constraints you are able to use a validation block within a variable to define specific rules for that variable. The condition argument is an expression that must return a true or false value.

```
variable "image_id" {
  type        = string
  description = "The id of the machine image (AMI) to use for the server."

  validation {
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"."
  }
}
```

- If the value returns false, Terraform will produce an error message that includes the error_message set.


### Assigning Values to Root Module Variables
- Variables declared in the root module can be set in a number of ways: CLI, Terraform Workspace, .tfvars files or as environment variables.

- If multiple values for the variables are found, Terraform will use the LAST VALUE it finds, overriding any previous values.

- Terraform loads variables in the following order:
    - Environment variables
    - 'terraform.tfvars' file if present
    - 'terraform.tfvars.json' file if present
    - '*.auto.tfvars' or '*.auto.tfvars.json' files if present
    - Any '-var' or '-var-file' option if specified

#### Variables on the CLI
- To specify variables on the command line, use the '-var' option:

```
terraform apply -var="image_id=ami-abc123"
terraform apply -var='image_id_list=["ami-abc123","ami-def456"]'
terraform apply -var='image_id_map={"us-east-1":"ami-abc123","us-east-2":"ami-def456"}'
```

#### Variable Definitions (.tfvars) Files
- To set lots of variables it is more convenient to specify their values in a definitions file and specify that on the CLI

```
terraform apply -var-file="testing.tfvars"
```

- These files use the same base syntax of Terraform but only consists of the variable name assignments:

```
image_id = "ami-abc123"
availability_zone_names = [
  "us-east-1a",
  "us-west-1c",
]
```

#### Environment Variables
- Variables named 'TF_VAR_' followed by the name of a declared variable is how you would define environment variables.








https://learn.hashicorp.com/terraform/certification/terraform-associate-study-guide 
https://www.terraform.io/docs/registry/modules/use.html