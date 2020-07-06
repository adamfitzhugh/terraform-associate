# Commands

### command: init
- 'terraform init' is used to initialise a working directory containing Terraform configuration files.

- By running a 'terraform init' with the '-upgrade' parameter youre able to manually force an update of the previously installed plugins. This doesn't happen by default.

### command: validate
- Validates the config files in a directory, referring only to the configuration and not accessing any remote services (such as state, provider API's etc.).

- Validate checks for config validity and ensures it's consistent regardless of what providers are used.

- Validate requires an initialised working directory with any referenced plugins and modules installed.

- A 'terraform plan' includes an implied validation check.

### command: plan
- Used to create an execution plan to determine what actions are required to achieve desired state.

- A plan is a convenient way to check that the execution plan matches what you expect without making any changes to resources or state.

### command: apply
- Used to apply the changes required to reach the desired state of the configuration.

### command: destroy
- Used to destroy Terraform-managed infrastructure.

### command: fmt
- Used to rewrite Terraform configuration files to a canonical format and style. The command applies Terraform language style conventions for readability purposes.

### command: taint
- Manually marks a Terraform-managed resource as tainted, forcing it to be destroyed and recreated on the next apply.

- This command will not modify infrastructure but will modify the state file in order to mark the specific resource as tainted. Once its marked as tainted, the next plan will show the resource to be destroyed and the next apply will implement this change.

- Note that tainting a resource may affect resources that depend on the newly tainted resource. For example a DNS resource that uses an IP address of a server may need to be modified to reflect the IP address of a tainted server.

### command: state
- Used for advanced state management. There are some cases you may need to amend Terraform state. Rather than modify the state directly, the 'terraform state' commands can be used in many cases instead.

### command: import
- Used to import existing resoures into Terraform

### command: refresh
- Used to reconcile the state Terraform knows about (via the state file) with real-world infrastructure. This can help detect any drift from the last known state.

- This does not modify infrastructure but can modify the state file itself.


## Terraform Workspaces
- Each Terraform config has an associated backend which defines how operations are executed and where state is stored. The persistent data stored in the backend belongs to a specific workspace.

- Certain backends support multiple workspaces allowing multiple states to be associated with a single configuration. The config still has only one backend, but multiple distinct instances of that config to be deployed without configuring a new backend.

### Using Workspaces
- Terraform starts with a single 'default' workspace. This cannot ever be deleted. 

- Workspaces are managed with the 'terraform workspace' set of commands.

- 'terraform workspace new <name>' - creates a new workspace.
- 'terraform workspace select <name>' - allows you to switch workspaces.

- Remember if you switch workspaces, when running a 'terraform plan', Terraform will not see any existing resources that existed in the other workspaces.

### Current Workspace Interpolation
- You are able to specify the name of the current workspace using the '${terraform.workspace} interpolation sequence. You should NOT use this though for remote operations against Terraform Cloud workspaces.

### When to use Multiple Workspaces
- Named workspaces allow for switching between multiple instances of a single configuration within a single backend. 

- A common use for workspaces is to create a parallel, distinct copy of a set of infrastructure in order to test changes before deploying in production.

- Non-default branches are often related to feature branches in version control. A developer may create a corresponding workspace to deploy into a temporary copy of the main infrastructure so that changes can be tested without affecting production.

- Named workspaces are not suitable for an isolation mechanism between production and staging environments. Instead, using re-useable modules to represent common elements. Using this method allows the backend configuration to only consist of a small number of module blocks.

### Workspace Internals
- Workspaces are technically equivalent to renaming your state file. Terraform wraps this up with a set of protections and support for remote state.

- Terraform stores the workspaces state in a directory called terraform.tfstate.d - the equivalent of the terraform.tfstate file for local-only deployments.

- For remote state, the workspaces are stored directly in the configured backend.