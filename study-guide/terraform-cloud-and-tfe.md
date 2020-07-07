# Terraform Cloud

## Terraform Cloud & TFE
- Terraform Cloud allows teams to use Terraform together. It gives these teams access to shared state and secret data for making changes to infrastructure, along with governing controls of Terraform configurations.

- Large enterprises can make use of Terraform Enterprise which is a self-hosted distribution of Terraform Cloud. It gives enterprises a private instance of Terraform Cloud with additional features such as audit logging and SAML authentication.

- For every workspace you have a 1:1 mapping between the configuration files and the state file.


## Workspaces
- Each Terraform configuration has an associated backend which describes how operations are executed and where state is stored. This persistent data is stored in a workspace.

- Certain backends support multiple named workspaces allowing multiple states to be associated with a single configuration.

### Using Workspaces
- Terraform starts with a single workspace named 'default' which cannot be deleted.

- Workspaces are managed with the 'terraform workspace' set of commands. When you create a new workspace for example, Terraform will not see any existing resources which existed on the default workspace as you are now in a new workspace. The resources still exist but are managed in another workspace.

### Current Workspace Interpolation
- Within Terraform configuration you are able to reference the current workspace using the '${terraform.workspace} interpolation sequence. You could use this for example as part of naming/tagging behaviour:

```
resource "aws_instance" "example" {
  tags = {
    Name = "web - ${terraform.workspace}"
  }

  # ... other arguments
}
```

### Multiple Workspaces
- An advantage of creating multiple workspaces is for production vs. non-production environments. You may require testing out a set of changes before deploying to a production environment.

- Non-default workspaces are often related to branches within version control, with the default workspace used as the 'master' branch.

- Once the branch code has been tested and verified, when it is merged into master branch or the default workspace, the test infrastructure can be destroyed and the temporary workspace deleted.

### Workspace Internals
- Workspaces are technically the same as renaming your state file.

- Terraform stores workspace states in a directory called 'terraform.tfstate.d'. This directory should be treated similarly to local-only 'terraform.tfstate'.

### Workspaces are Collections of Infrastructure
- When ran locally, Terraform manages each collection of infrastructure with a persistent working directory which includes its configuration, state data and variables. This means you can organise your infrastructure resources into meaningful groups by keeping multiple directories. Terraform Cloud swaps these multiple directories into workspaces.

- Each workspace retains backups of previous state files. This allows you to view state changes over time.

- Terraform Cloud also retains a record of all run activity including summaries, logs, change references etc.

### Planning and Organising Workspaces
- Terraform encourages you to split up your workspaces into smaller chunks to make them more manageable. For example, the code which manages your production environment could be split into networking (networking-prod), main applications configuration (app1-prod), monitoring configuration (monitoring-prod) etc.

- Using this method allows you to re-use configurations in a dev environment, such as app1-dev workspace etc.


## Private Module Registry
- Terraform Cloud's private module registry allows you to share modules across your organisation. It includes versioning support and a configuration designer to help you build new workspaces faster.

### Using Private Modules
- All users in your organisation are able to view private modules by looking in the Modules section on Terraform Cloud.

- Public registry uses the following syntax: <NAMESPACE>/<MODULE NAME>/<PROVIDER>

- Private module syntax reference is <HOSTNAME>/<ORGANISATION>/<MODULE NAME>/<PROVIDER>

```
module "vpc" {
  source  = "app.terraform.io/example_corp/vpc/aws"
  version = "1.0.4"
}
```

- If you're using a SaaS version of Terraform Cloud the hostname is 'app.terraform.io', the second part is the namespace of your organisation.

### Running Configurations with Private Modules
- In Terraform Cloud you can use private modules during plans and applied as long as the workspace is configured to use Terraform 0.11 or higher.

- If you're using the CLI you also need to use Terraform 0.11 or higher but you must provide a valid Terraform Cloud API token.

### Publishing Modules to Terraform Cloud Private Module Registry
- The private module registry lets you public Terraform modules to be consumed across your organisation. It works very similar to the public Terraform registry, except that it uses your VCS integrations instead of looking for public GitHub repositories.

- Once you have created a new module in your VCS, the registry can automatically detect a lot of the information it needs to publish it, such as modules name and available versions.

- Module repositories must use the name format: terraform-<PROVIDER>-<NAME> in order for it to become available for use.

- The registry uses VCS tags to control module versions. Once a new tag has been created in VCS, the registry will automatically import to the new version.

### Configuration Designer
- Terraform Cloud's private module registry includes a configuration designer which can help you spend less time writing code in a module-centric workflow.

- The configuration designer is like an interactive documentation for your private modules and saves you time by auto-discovering your variables and searching modules and workspace outputs for reusable variables.

- Once finished a main.tf file is created which you must create a new VCS repo for. The designer does not auto-create any repos or workspaces.


## Sentinel
- Treating policy as code requires a way to specify policies and a mechanism to enforce them. Sentinel provides a simple policy-orientated language to write policies which integrates with tools like TFE.

- The below extract shows staging is deployed to "us-east-1" while production is deployed to "us-west-1"

```
import "tfplan"

valid_regions = {"production": "us-west-1", "staging": "us-east-1"}
env_region = valid_regions[tfplan.variables.env]

is_not_aws = func(type) {
    return type is not "aws"
}

// Check the provider alias region matches the environment region
validate_aws_region = func(provider) {
    return all provider.alias as a {
        a.config.region is env_region
    }
}

main = rule {
    all tfplan.config.providers as type, provider {
        is_not_aws(type) or validate_aws_region(provider)
    }
}
```

- Sentinel also has a full testing framework. This allows you to create multiple test cases for pass and fail scenarios, allowing the policy to be easily validated without having to execute it.

- Once wrote the policy can automatically be enforced without manual review. It is best to do this in the change path so violations are prevented rather than just detected.

- Sentinel involves:
    - Defining policies
    - Managing policies for organisations
    - Enforcing policy checks on runs
    - Mocking Sentinel Terraform data