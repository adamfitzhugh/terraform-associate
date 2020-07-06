# Manage State

## State Locking
- To help corruption of state, Terraform will lock state from operations which could write state.

- Locking happens automatically on any operation that could write state.

- You can disable this behaviour by using the '-lock' parameter but it is not recommended.

- Rememeber not all backends support locking.

### Force Unlock
- If unlocking fails, you are able to use the 'force-unlock' command to force it. However, be careful with this as it could cause multiple writers if someone else is holding the lock.

- To protect you, 'force-unlock' command requires a unique lock ID.


## Sensitive Data in State
- Terraform state can also contain sensitive data (including passwords) depending on the resources you configure.

- When using remote state, state is only ever held in memory when used by Terraform. You are able to encrypt this at rest.

### Recommendations
- Since version 0.9 Terraform does not persist state to the local disk when remote state is in use, and you can encrypt state data at rest with some backends. For example:
    - Using Terraform Cloud always encrypts state data at rest.
    - S3 backend supports encryption at rest when the 'encrypt' option is enabled.


## Backends
- A backend determines how state is loaded to aid in an apply. By default, this backend is set to local.

- Benefits of backends:
    - Working in a Team allows multiple individuals to be working on a Terraform project by using a remote state.
    - Keeping sensitive data off local disk.
    - Some backends support remote operations which allows the execution to be ran remotely, meaning if you turned your machine off the operation would still complete.

- Example of a local backend:

```
terraform {
  backend "local" {
    path = "relative/path/to/terraform.tfstate"
  }
}
```

```
data "terraform_remote_state" "foo" {
  backend = "local"

  config = {
    path = "${path.module}/../../terraform.tfstate"
  }
}
```


## Backend Configuration
- Once a backend is defined it must be initialised before use.

- When using a backend for the first time, Terraform will give you the option to migrate your state to a new backend, which allows you to adopt backends without losing current state.

- Terraform recommends manually backing up state too by copying the 'terraform.tfstate' file to another location.

### Partial Configuration
- You do not need to specify every argument in backend configuration. Omitting certain arguments (such as passwords or keys) is known as partial configuration.

- When using partial configuration the remaining arguments must be provided as part of the initialisation process. You can supply these through one of the following:
    - Interactive inputs on the CLI.
    - From a configuration file using the '-backend-config=PATH' option.
    - Key/value pairs can be specified via the 'init' command and adding the '-backend-config=KEY=VALUE' option.

- Command line options override settings in the main configuration.

- The final merged configuration is stored on disk in the .terraform directory which should be ignored from version control.

- You must at least specify an empty backend configuration when using partial configuration as shown below:

```
terraform {
  backend "consul" {}
}
```

- You can also specify the settings on the CLI if you prefer:

```
$ terraform init \
    -backend-config="address=demo.consul.io" \
    -backend-config="path=example_app/terraform_state" \
    -backend-config="scheme=https"
```

### Changing Configuration
- You can change your backend configuration at any time, whether thats the configuration itself or the type of backend. This must be followed by a new initialisation.

- As part of the new initialisation, Terraform will ask if you want to migrate your existing state to the new configuration - this allows you to switch between backends.

- If you're just reconfiguring the same backend, Terraform will still ask but you can just reply 'no' to this.

### Unconfiguring a Backend
- You can simply remove a backend from the configuration file if you no longer want to use it.

- If you do this, when you reinitialise, Terraform will ask if you'd like to migrate your state back down to normal local state.