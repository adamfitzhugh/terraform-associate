# Master the Workflow

The core Terraform workflow has 3 steps:

1. Write - Author of the IaC
2. Plan - Preview changes before applying
3. Apply - Provision reproducible infrastructure


## Working as an Individual

### Write
- Repeatedly running plans against your work helps you flush out syntax errors and ensure your config is correct.

### Plan
- Because 'terraform apply' will display a plan of confirmation before proceeding, thats the command you run for final review.

### Apply
- Once your code is applied and working, it's common to push your version control repository to a remote location for safekeeping.


## Working as a Team

### Write
- Individuals can save their changes in branches to avoid colliding with each other's work.

- As time and the team grows, so does the number of sensitive input variables. In order to avoid security breaches, the larger the team means it makes sense to migrate to a CI environment for Terraform operations.

### Plan
- Within teams, Terraform's plan output creates an opportunity for team members to review each others work. This allows the team to catch mistakes before any changes are made. This normally happens alongside pull requests within version control - the individual can then evaluate whether the intent of the change is being achieved by the plan.

- In addition to reviewing a plan, the team can also decide when this change can happen. If they notice a certain change involves disruption, they may delay merging the pull request until they can schedule a maintenance window.

### Apply
- With an apply it's important that the final concrete plan is reviewed as it may be different than the one reviewed on the pull request due to issues like merge order or infrastructure changes.

- At this point the usual questions are asked. Is there a service outage with this change? Is any part of this change high risk? Should we be monitoring this change?


## The Core Workflow Enhanced by Terraform Cloud
While the above describes the typical workflow for an organisation, this can be streamlined by using tools such as Terraform Cloud.

### Write
- Terraform Cloud offers a centralised and secure location to store input variables and state by the writer creating a terraform backend piece of code.

- Once the backend config is wrote, all that is required by the teams to use it is a Terraform Cloud API key.

```
$ terraform workspace select my-app-dev
Switched to workspace "my-app-dev".

$ terraform plan

Running plan remotely in Terraform Enterprise.

Output will stream here. To view this plan in a browser, visit:

https://app.terraform.io/my-org/my-app-dev/.../

Refreshing Terraform state in-memory prior to plan...

# ...

Plan: 1 to add, 0 to change, 0 to destroy.
```

### Plan
- Once a pull request is created and ready to be reviewed, Terraform Cloud automatically runs a plan along with status updates of the plan.

- Once the plan is complete, the status update indicates whether there were any changes in the plan right from the pull request. A teammate can then review this request by viewing the raw log and then approve the change.

- Once the plan has been approved, Terraform Cloud performs the apply.