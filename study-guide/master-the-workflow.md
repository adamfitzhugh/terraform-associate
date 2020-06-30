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