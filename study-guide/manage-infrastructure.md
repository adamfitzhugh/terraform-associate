# Manage Infrastructure

## Providers
- Behaviours of resources rely on associated resource types known as providers.

- Each provider offers a set of named resource types and defines which arguments each resource accepts

- Resource types belonging to the same provider share the same configuration, avoiding the need to repeat this information across every resource declaration.

### Provider Configuration
Example provider configuration:

```
provider "google" {
  project = "acme-app"
  region  = "us-central1"
}
```

- Terraform associates each resource type with a provider by taking the first word of the resource type name, so "google" is assumed to be the provider of the resource google_compute_instance

- The body of the block defines arguments for the provider itself

- 