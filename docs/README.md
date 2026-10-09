# Terraform Study Guide

This guide contains the theory behind the hands-on Terraform labs. Read the chapters in order for a structured path, or use the topic that matches the problem you are solving.

## Recommended order

1. [IaC Concepts](01-iac-concepts.md)
2. [Terraform Basics](02-terraform-basics.md)
3. [Core Terraform Workflow](03-core-workflow.md)
4. [Terraform State](04-state.md)
5. [Terraform Configuration](05-configuration.md)
6. [Modules and Collaboration](06-modules.md)
7. [Vault, HCP Terraform, and Terraform Enterprise](07-hcp-terraform.md)

The [practical index](../practice/README.md) connects each topic to the labs that exercise it.

For background on how the material is developed and checked, read [How I Use AI in This Project](ai-workflow.md). This is a project note, not an additional Terraform chapter.

## Topic index

Use the links below to jump directly to a topic. You can also use your browser's search (`Ctrl+F`, or `Cmd+F` on macOS) to find a term such as `locals`, `precondition`, or `TF_VAR` on this page.

### 1. IaC Concepts

- [What Infrastructure as Code means](01-iac-concepts.md#what-infrastructure-as-code-means)
- [First Terraform run](01-iac-concepts.md#how-terraform-works-a-first-run)
- [Configuration, state, and remote objects](01-iac-concepts.md#configuration-state-and-remote-objects)
- [Reading plan symbols](01-iac-concepts.md#reading-terraform-plan-symbols)

### 2. Terraform Basics

- [Providers and the Terraform Registry](02-terraform-basics.md#providers)
- [Multiple provider configurations and aliases](02-terraform-basics.md#multiple-provider-configurations)
- [Terraform settings block](02-terraform-basics.md#terraform-settings-block-terraform-)
- [Version constraints](02-terraform-basics.md#terraform-versioning)
- [Dependency lock file](02-terraform-basics.md#dependency-lock-file-terraformlockhcl)
- [Resource blocks and references](02-terraform-basics.md#resource-blocks-and-references)
- [Project layout and comments](02-terraform-basics.md#common-project-layout-and-comments)
- [Data types](02-terraform-basics.md#data-types)
- [Objects and sets](02-terraform-basics.md#complex-data-types-object-and-set)
- [Input variables and outputs](02-terraform-basics.md#variables-and-output-values)
- [Environment variables (`TF_VAR_`)](02-terraform-basics.md#assigning-input-values-through-environment-variables)
- [Variable assignment precedence](02-terraform-basics.md#variable-assignment-precedence-lowest-to-highest)

### 3. Core Terraform Workflow

- [Core CLI commands](03-core-workflow.md#core-cli-commands)
- [Saving and inspecting plans (`-out`)](03-core-workflow.md#saving-and-inspecting-execution-plans)
- [Querying outputs](03-core-workflow.md#querying-outputs)
- [Resource targeting (`-target`)](03-core-workflow.md#resource-targeting--target)
- [Resource replacement and dependency graphs](03-core-workflow.md#resource-replacement-and-dependency-visualization)
- [Logging and debugging (`TF_LOG`)](03-core-workflow.md#terraform-logging--debugging-tf_log)
- [API throttling and concurrency](03-core-workflow.md#performance-optimization-api-throttling)

### 4. Terraform State

- [What Terraform state is (`terraform.tfstate`)](04-state.md#what-terraform-state-is)
- [Risks of storing state in Git](04-state.md#security-risk-of-storing-terraform-state-in-git)
- [Local and remote backends](04-state.md#terraform-backend)
- [State locking and force unlock](04-state.md#state-locking)
- [Backend configuration and state migration](04-state.md#backend-configuration)
- [S3 backend](04-state.md#s3-backend)
- [CLI workspaces](04-state.md#terraform-workspaces-environment-management)
- [State management commands](04-state.md#terraform-state-management-commands)
- [State refactoring (`moved`)](04-state.md#state-refactoring-moved-blocks)
- [Importing existing resources](04-state.md#terraform-import)
- [Removing state bindings (`removed`)](04-state.md#removed-blocks)
- [Sharing outputs through remote state](04-state.md#cross-project-collaboration-using-remote-state-data-source)

### 5. Terraform Configuration

- [String templates and interpolation](05-configuration.md#string-templates-and-interpolation)
- [Local values (`locals`)](05-configuration.md#local-values-locals)
- [Data sources (`data`)](05-configuration.md#data-sources-data-blocks)
- [Functions and expressions](05-configuration.md#essential-terraform-functions-and-expressions)
- [Resource repetition (`count`)](05-configuration.md#the-count-meta-argument)
- [Key-based resource identity (`for_each`)](05-configuration.md#advanced-looping-for_each-vs-count)
- [Splat expressions](05-configuration.md#splat-expressions-)
- [Dynamic nested blocks](05-configuration.md#dynamic-blocks)
- [Resource dependencies (`depends_on`)](05-configuration.md#resource-dependencies)
- [Lifecycle rules](05-configuration.md#the-lifecycle-meta-argument)
- [Input variable validation](05-configuration.md#input-variable-validation)
- [Preconditions and postconditions (`precondition`, `postcondition`)](05-configuration.md#custom-conditions-precondition--postcondition)
- [Check blocks](05-configuration.md#check-blocks-and-continuous-validation)
- [Provisioners and connections](05-configuration.md#provisioners-the-last-resort)
- [Security practices](05-configuration.md#terraform-security-primer)
- [Sensitive variables and outputs](05-configuration.md#sensitive-input-variables-and-outputs)
- [Ephemeral values and write-only arguments](05-configuration.md#ephemeral-values-and-write-only-arguments)

### 6. Modules and Collaboration

- [Root and child modules](06-modules.md#terraform-modules)
- [Module directory structure](06-modules.md#standard-module-structure-hashicorp-best-practices)
- [Module sources and versions](06-modules.md#module-sources)
- [Passing inputs and variable scope](06-modules.md#passing-variables-into-a-module)
- [Reading child outputs](06-modules.md#extracting-outputs--cross-referencing-resources)
- [Provider inheritance and alias mapping](06-modules.md#the-provider-rule-exam-trap)
- [Terraform tests](06-modules.md#terraform-tests)
- [Choosing public modules](06-modules.md#choosing-the-right-public-module)
- [Publishing to the Terraform Registry](06-modules.md#requirements-for-publishing-to-the-terraform-registry)
- [Git collaboration](06-modules.md#git-for-team-collaboration-with-terraform)
- [Terraform `.gitignore`](06-modules.md#terraform-and-gitignore)

### 7. Vault, HCP Terraform, and Terraform Enterprise

- [Vault basics](07-hcp-terraform.md#hashicorp-vault-basics-and-why-vault-matters)
- [Terraform and Vault integration](07-hcp-terraform.md#terraform-and-vault-integration)
- [HCP Terraform and Terraform Enterprise](07-hcp-terraform.md#hashicorp-cloud-platform-hcp-terraform-and-terraform-enterprise)
- [Organizations, projects, and workspaces](07-hcp-terraform.md#core-structure)
- [HCP versus CLI workspaces](07-hcp-terraform.md#hcp-terraform-workspaces-vs-cli-workspaces)
- [Remote, local, and agent execution](07-hcp-terraform.md#execution-modes)
- [VCS, CLI, and API workflows](07-hcp-terraform.md#run-workflows)
- [Connecting with the `cloud` block](07-hcp-terraform.md#connecting-with-the-cloud-block)
- [Variables, credentials, and variable sets](07-hcp-terraform.md#variables-and-credentials)
- [Collaboration and governance](07-hcp-terraform.md#collaboration-and-governance)
- [Sentinel policies](07-hcp-terraform.md#sentinel-policy-as-code)
- [Run triggers and run tasks](07-hcp-terraform.md#automating-workspace-runs-and-external-checks)
- [Explorer](07-hcp-terraform.md#explorer)
- [Air-gapped Terraform Enterprise](07-hcp-terraform.md#air-gapped-terraform-enterprise)
- [Editions and pricing](07-hcp-terraform.md#editions-and-pricing)
- [Managed-resource billing](07-hcp-terraform.md#managed-resource-billing-concept)
