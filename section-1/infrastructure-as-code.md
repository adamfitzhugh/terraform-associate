# Infrastructure as Code Concepts

## 1a. Explain what IaC is
Described using a high level configuration syntax. It allows your infrastructure to be versioned and treated as code. This also allows infrastructure to be shared and re-used.

IaC can be applied throughout the lifecycle on both initial build and throughout the life of the infrastructure. Day 0 is defined as provisioning your initial infrastructure.

Day 1 refers to OS and application configurations you apply after you've initially built your infrastructure.

IaC makes it easy to provision and apply infrastructure changes, saving time. It also standardizes workflows across different infrastructure vendors by using a common syntax across them.

IaC makes changes idempotent, consistent, repeatable and predictable. Without IaC, scaling up infrastructure would need to be done manually which could be an error prone approach and make the deployment inconsistent. This could then affect performance, usability and security.

IaC also allows us to review the results of the code before it is applied to our target environments.

### Terraform Workflows
1. Scope - confirms what resources need to be created for a given project.
2. Author - create the configuration based on the scoped parameters
3. Initialise - 'terraform init' downloads the correct provider plug-ins for the project
4. Plan & Apply - 'terraform plan' verifies what is to be applied. 'terraform apply' creates the resources as well at a state file which compares future changes with what already exists in your environment

### Advantages of Terraform
1. Platform Agnostic
2. State Management
3. Operator Confidence (by planning and applying config)

## 1b. Describe 