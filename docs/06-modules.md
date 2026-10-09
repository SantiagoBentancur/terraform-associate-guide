# Modules and Collaboration

Reusable modules, module inputs and outputs, provider rules, sources, testing, publishing, and team collaboration.

## Terraform Modules

According to the official HashiCorp documentation, **a module is a container for resources that are used together.** Modules package related infrastructure configuration behind a reusable interface.

Every Terraform configuration has at least one module:

* **Root module:** The `.tf` files in the working directory where Terraform runs.
* **Child module:** A module called from another module using a `module` block.
* **Calling module:** The module containing that `module` block. A calling module can be the root module or another child module.

Modules can call other modules, but HashiCorp recommends keeping the module tree relatively flat. Excessive nesting and unnecessary wrappers can make configurations harder to understand and reuse. See HashiCorp's official [module development guidance](https://developer.hashicorp.com/terraform/language/modules/develop).

### The Module Data Flow Diagram

This chapter builds a small child module that creates one VPC. The root module supplies its CIDR block and reads the resulting VPC ID.

Input variables and output values form the child module's public interface:

```text
Root module                          Child module: ./modules/vpc
var.network_cidr
    │
    └─ module "network"
         vpc_cidr ──────────────────> var.vpc_cidr
                                          │
                                     aws_vpc.main
                                          │
module.network.vpc_id <────────────── output "vpc_id"
```

The next sections build this example step by step.

### Standard Module Structure (HashiCorp Best Practices)
For reusable modules, HashiCorp recommends the following minimal structure:

```text
.
├── README.md
├── main.tf
├── variables.tf
└── outputs.tf
```

These filenames are organizational conventions understood by people and documentation tooling. Terraform evaluates all `.tf` files in the module directory together, and the root module directory is the only strictly required element of the standard structure.

* **`main.tf`:** The primary entry point; commonly contains resources and nested module calls.
* **`variables.tf`:** Input variable declarations with descriptions and appropriate type constraints.
* **`outputs.tf`:** Output declarations with descriptions.
* **`README.md`:** The module's purpose, usage, prerequisites, and important behavior.

Larger reusable modules may also include:

* **`examples/`:** Complete examples showing how callers use the module.
* **`modules/`:** Nested modules; a nested module with its own README is considered externally usable.
* **`LICENSE`:** Strongly recommended for publicly distributed modules.

See the official [standard module structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure).

Our example uses this layout. The root is the directory where you run Terraform; `modules/vpc` contains the child:

```text
example/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── versions.tf
└── modules/
    └── vpc/
        ├── README.md
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        ├── versions.tf
        └── tests/
            └── vpc.tftest.hcl
```

The child's README should describe its purpose, inputs, outputs, and usage. Its `versions.tf` declares provider requirements; provider configuration stays in the root.

### Module Sources
The required `source` argument tells Terraform where to retrieve a child module. Common sources include:

* **Local path:** `source = "./modules/vpc"`. A local relative module path must begin with `./` or `../`.
* **Terraform Registry:** `source = "terraform-aws-modules/vpc/aws"`.
* **Version control:** `source = "git::https://github.com/example/terraform-vpc.git?ref=v1.2.0"`.

The local VPC example uses `source = "./modules/vpc"`. The following Registry example is an alternative source with its own input interface, rather than an additional block for the local example. Constrain Registry module versions to avoid unexpected upgrades:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 6.0"

  name = "application"
  cidr = "10.0.0.0/16"
}
```

The `version` argument applies only to Registry modules. Git sources use the `ref` query parameter to select a branch, tag, or commit. Prefer an immutable release tag or full commit hash instead of an unpinned default branch.

Run `terraform init` after adding or changing a module source or version. Use `terraform init -upgrade` when you want Terraform to reconsider installed versions and select newer versions allowed by the configured constraints. See the official [module source documentation](https://developer.hashicorp.com/terraform/language/modules/configuration).

### Passing Variables into a Module

Declare the child module's input in `modules/vpc/variables.tf`:

```hcl
variable "vpc_cidr" {
  description = "IPv4 CIDR block assigned to the VPC"
  type        = string
}
```

Use that input to create the VPC in `modules/vpc/main.tf`:

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
}
```

In the root directory, declare a separate input in `variables.tf`:

```hcl
variable "network_cidr" {
  description = "IPv4 CIDR block passed to the network module"
  type        = string
  default     = "10.0.0.0/16"
}
```

Call the child module from the root's `main.tf`:

```hcl
module "network" {
  source = "./modules/vpc"

  vpc_cidr = var.network_cidr
}
```

Read `vpc_cidr = var.network_cidr` as: pass the root's `network_cidr` value into the child's `vpc_cidr` input. The argument on the left must match the child variable declaration.

Variables are scoped to their own module. In the root, `var.network_cidr` refers to the root variable; inside the child, `var.vpc_cidr` refers to the child variable. Giving variables identical names would not connect them automatically—the assignment in the `module` block makes the connection.

Root `.tfvars` files and `TF_VAR_*` environment variables supply root inputs. The root then passes those values to the child through the module call. A child variable without a default requires a corresponding argument; a child variable with a default makes that argument optional.

### Extracting Outputs & Cross-Referencing Resources

A calling module accesses a child's results through output blocks. It cannot directly reference the child's resource addresses.

In `modules/vpc/outputs.tf`, expose the VPC ID and CIDR:

```hcl
output "vpc_id" {
  description = "ID of the VPC created by this module"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "IPv4 CIDR block configured on the VPC"
  value       = aws_vpc.main.cidr_block
}
```

The root uses `module.<MODULE_NAME>.<OUTPUT_NAME>`. Add this output to the root's `outputs.tf`:

```hcl
output "network_vpc_id" {
  description = "ID of the VPC returned by the network module"
  value       = module.network.vpc_id
}
```

To use that VPC in another root resource, add the following to the root's `main.tf`:

```hcl
resource "aws_security_group" "application" {
  name_prefix = "application-"
  description = "Application security group"
  vpc_id      = module.network.vpc_id
}
```

The reference connects the security group to the child's VPC and lets Terraform infer the dependency. The child output `vpc_id` is available to the root configuration; the root output `network_vpc_id` exposes it through commands such as `terraform output network_vpc_id` after an apply.

### The Provider Rule (Exam Trap)

Provider configurations belong in the root module. Reusable child modules declare provider requirements and receive configurations from their caller.

For this example, create `modules/vpc/versions.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 6.0"
    }
  }
}
```

Create `versions.tf` in the root directory:

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

Create `providers.tf` in the root directory:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

The VPC module automatically inherits this default AWS configuration. The child requirement identifies the provider it needs; it does not configure a region or credentials. Reusable children normally declare a minimum supported provider version, while the root controls the overall version-selection policy.

With AWS credentials available, run `terraform init`, `terraform validate`, and `terraform plan` from the root directory to review the VPC and security group example. Applying the plan creates AWS resources; use `terraform destroy` from the same root directory to clean them up when finished.

#### Passing an Aliased Provider

Aliased configurations are passed explicitly. To deploy the same VPC module in another region, add an alias to the root's `providers.tf`:

```hcl
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```

Update the existing root module call:

```hcl
module "network" {
  source = "./modules/vpc"

  vpc_cidr = var.network_cidr

  providers = {
    aws = aws.west
  }
}
```

Here, the child still uses its default local provider name `aws`, but the caller supplies the root's `aws.west` configuration. The child does not need `configuration_aliases` for this mapping.

If a child itself refers to an aliased name such as `aws.secondary`, it must declare that name in `configuration_aliases`, and the caller must map it explicitly. That is a different case from passing a root alias to the child's default `aws` name. [Lab 15](../practice/lab_15/lab_15.md) practices multiple provider names within a child module.

For the following example test, use the original default-provider module call. See the official [providers within modules documentation](https://developer.hashicorp.com/terraform/language/modules/develop/providers).

### Terraform Tests

Terraform 1.6 and later can execute test files ending in `.tftest.hcl`. We will test the VPC child directly, independently of the root module and its security group.

Create `modules/vpc/tests/vpc.tftest.hcl`:

```hcl
provider "aws" {
  region = "us-east-1"
}

run "preserves_requested_cidr" {
  command = plan

  variables {
    vpc_cidr = "10.20.0.0/16"
  }

  assert {
    condition     = output.vpc_cidr == "10.20.0.0/16"
    error_message = "The VPC module must configure and return the requested CIDR."
  }
}
```

This test supplies the child's input and checks the output derived from `aws_vpc.main.cidr_block`. If the module accidentally hardcodes another CIDR instead of using `var.vpc_cidr`, the assertion fails. It checks the planned configuration, not AWS connectivity or the behavior of a deployed VPC.

The test's provider block supplies an AWS configuration for the standalone test run. The child continues to declare only provider requirements in its normal `.tf` files.

From the repository root, initialize and test the child:

```bash
terraform -chdir=modules/vpc init
terraform -chdir=modules/vpc test
```

These paths refer to the example directory you created. `command = plan` avoids creating the VPC, but the real AWS provider may still require credentials and access AWS APIs. A run with `command = apply` creates test infrastructure and can incur costs.

To select this test file only:

```bash
terraform -chdir=modules/vpc test -filter=tests/vpc.tftest.hcl
```

| Command | Purpose |
| :--- | :--- |
| `terraform validate` | Check syntax and internal configuration consistency |
| `terraform plan` | Preview changes for the current configuration and state |
| `terraform test` | Execute test runs and assertions |

Normal `validate`, `plan`, and `apply` commands do not execute these test files. See HashiCorp's [Terraform test documentation](https://developer.hashicorp.com/terraform/language/tests).

---

### Choosing the Right Public Module
When sourcing modules from the public Terraform Registry:
* Consider **Partner modules** when appropriate. Partner modules are reviewed by HashiCorp and expected to be actively maintained by HashiCorp partners.
* A Partner badge does not guarantee that a module has more features or is the best design for your architecture.
* Review ownership, maintenance history, recent releases, documentation, supported Terraform and provider versions, release practices, open issues, and security implications before adopting any module.

See HashiCorp's official [Partner module guidance](https://developer.hashicorp.com/terraform/registry/modules/partner).

### Requirements for Publishing to the Terraform Registry
The following requirements apply specifically to modules published in the **public** Terraform Registry. Private registries use different publishing workflows.

1. **GitHub Repository:** The code must be hosted on a public GitHub repository.
2. **Strict Naming Convention:** The repository *must* be named exactly: `terraform-<PROVIDER>-<NAME>` *(Example: `terraform-aws-webserver`)*.
3. **Repository Description:** The GitHub repository must have a short description.
4. **Standard Module Structure:** The repository must follow Terraform's standard module structure.
5. **Semantic Version Tags:** At least one release tag must use `x.y.z` syntax, optionally prefixed with `v` (e.g., `v1.0.0`).

See the official [public module publishing requirements](https://developer.hashicorp.com/terraform/registry/modules/publish).

---
## Git for Team Collaboration with Terraform

Git allows multiple engineers to collaborate on the same Terraform codebase safely.

### What Should Be Committed
Commit the Terraform configuration files that describe the desired infrastructure:
* `main.tf`
* `variables.tf`
* `outputs.tf`
* `providers.tf`
* `modules/`
* `.terraform.lock.hcl`
* Example variable files such as `dev.tfvars.example`

### What Should Not Be Committed
Do **not** commit files that contain local machine data, downloaded providers, secrets, or Terraform state:
* `.terraform/`
* `terraform.tfstate`
* `terraform.tfstate.backup`
* `*.tfvars` files if they contain secrets
* Crash logs and local override files

### Terraform and `.gitignore`

A `.gitignore` file prevents Git from tracking local Terraform files that should stay out of the repository.

#### Common Terraform `.gitignore`
```gitignore
# Local Terraform provider/plugins directory
.terraform/

# Terraform state files
*.tfstate
*.tfstate.*

# Crash logs
crash.log
crash.*.log

# Sensitive variable files
*.tfvars
*.tfvars.json

# Local override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# CLI config files
.terraformrc
terraform.rc
```

> **Important:** If your team uses non-sensitive shared variable files, commit an example file such as `dev.tfvars.example`, not the real secret file.

---

### Basic Team Workflow
```bash
git checkout -b feature/add-network
terraform fmt
terraform validate
terraform plan
git status
git add main.tf variables.tf outputs.tf
git diff --staged
git commit -m "Add network module"
git push origin feature/add-network
```

The team can then review the pull request before the Terraform change is applied.

---
