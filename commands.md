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
- 