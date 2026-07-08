\# Terraform Fundamentals



\## What it is

Terraform is an Infrastructure as Code (IaC) tool: you describe the

infrastructure you want in `.tf` files (declarative), and Terraform figures

out how to make reality match. It talks to cloud APIs (AWS, GCP, Azure) and

hundreds of other providers through a plugin system.



The core idea: \*\*your infrastructure lives in git, not in a web console.\*\*



\## The mental model

```mermaid

flowchart LR

&#x20;   TF\[.tf files: desired state] --> Plan\[terraform plan]

&#x20;   State\[(terraform.tfstate: current state)] --> Plan

&#x20;   Plan --> Diff\[Diff: what will change]

&#x20;   Diff --> Apply\[terraform apply]

&#x20;   Apply --> Cloud\[Cloud API]

&#x20;   Cloud --> State

```



Terraform's job: read desired state (`.tf`) + current state (`.tfstate`) →

compute the diff → make API calls to close the gap.



\## The core workflow

```

terraform init      → download providers, set up backend

terraform fmt       → format files

terraform validate  → syntax check

terraform plan      → show what will change (no changes made)

terraform apply     → make the changes

terraform destroy   → tear it all down

```



Golden rule: \*\*always read the plan before applying.\*\* Every senior engineer

has a war story about skipping this.



\## Core concepts (the vocabulary)

\- \*\*Provider\*\* — plugin for a specific API (`aws`, `google`, `azurerm`, `github`, `cloudflare`, etc.).

\- \*\*Resource\*\* — a thing you manage (`aws\_instance`, `aws\_s3\_bucket`).

\- \*\*Data source\*\* — a thing you \*read\* but don't manage (`data "aws\_ami" "ubuntu"`).

\- \*\*Variable\*\* — input to your config (`var.region`).

\- \*\*Output\*\* — value exposed after apply (`output "instance\_ip"`).

\- \*\*Local\*\* — a computed value used inside the config (`local.name\_prefix`).

\- \*\*Module\*\* — a reusable bundle of resources. Every Terraform project is technically a module.

\- \*\*State\*\* — the file recording what Terraform manages and its current attributes.

\- \*\*Backend\*\* — where the state file lives (local, S3, Terraform Cloud, GCS).



\## A minimal example

```hcl

terraform {

&#x20; required\_providers {

&#x20;   aws = { source = "hashicorp/aws", version = "\~> 5.0" }

&#x20; }

}



provider "aws" {

&#x20; region = var.region

}



variable "region" {

&#x20; type    = string

&#x20; default = "eu-west-1"

}



resource "aws\_s3\_bucket" "notes" {

&#x20; bucket = "devops-hale-notes-${random\_id.suffix.hex}"

}



resource "random\_id" "suffix" {

&#x20; byte\_length = 4

}



output "bucket\_name" {

&#x20; value = aws\_s3\_bucket.notes.bucket

}

```



\## State — the thing you must understand

The state file is Terraform's memory. It maps `.tf` resources to real cloud IDs.



\- \*\*Local state\*\* (`terraform.tfstate` on your laptop) — fine for learning, terrible for teams.

\- \*\*Remote state\*\* (S3 + DynamoDB lock) — the standard for teams: shared, versioned, locked to prevent concurrent applies.



State contains \*\*plaintext sensitive values\*\* (passwords, keys). Encrypt at

rest. Never commit `terraform.tfstate` to git.



\## Remote state (S3 + DynamoDB) snippet

```hcl

terraform {

&#x20; backend "s3" {

&#x20;   bucket         = "my-tf-state"

&#x20;   key            = "devops-hale/terraform.tfstate"

&#x20;   region         = "eu-west-1"

&#x20;   dynamodb\_table = "tf-state-lock"

&#x20;   encrypt        = true

&#x20; }

}

```



\## Modules — write once, reuse everywhere

A module is just a folder with `.tf` files. Call it like a function:



```hcl

module "vpc" {

&#x20; source  = "terraform-aws-modules/vpc/aws"

&#x20; version = "5.1.2"



&#x20; name = "prod-vpc"

&#x20; cidr = "10.0.0.0/16"

&#x20; azs  = \["eu-west-1a", "eu-west-1b"]

}

```



The registry (registry.terraform.io) has battle-tested modules for most AWS

services. Don't reinvent the VPC module — use the one everyone else uses.



\## The dependency graph

Terraform builds a DAG (directed acyclic graph) of resources from your `.tf`

files. It uses implicit dependencies (referencing an attribute like

`aws\_vpc.main.id` creates one) and explicit dependencies (`depends\_on = \[...]`).



Then it applies resources \*\*in parallel\*\* where possible, in order where required.



\## Common gotchas

\- \*\*Never edit resources managed by Terraform through the console.\*\* State drifts. Import or destroy-and-recreate.

\- \*\*Provider version drift\*\* — pin versions with `\~>` or exact. Otherwise `terraform init` on a coworker's machine gets a different provider.

\- \*\*State locks stuck after a crash\*\* — `terraform force-unlock <lock-id>`.

\- \*\*Secrets in `.tf` files\*\* — never. Use env vars, AWS Secrets Manager, or Vault.

\- \*\*Big monolithic state\*\* — every plan gets slower. Split by environment (dev/stage/prod) and by concern (network / compute / data).

\- \*\*`count` vs `for\_each`\*\* — prefer `for\_each` (map/set) over `count` (list). Adding an item at index 0 with `count` reshuffles everything.

\- \*\*`terraform destroy` in prod\*\* — it does exactly what it says. Use workspaces or separate state files per environment to keep blast radius small.



\## Terraform vs alternatives

| Tool | Language | Notes |

|---|---|---|

| Terraform | HCL (declarative) | Cloud-agnostic, huge ecosystem |

| OpenTofu | HCL (declarative) | Open-source fork of Terraform |

| Pulumi | Real languages (TS/Python/Go) | Great for developers, smaller ecosystem |

| CloudFormation | YAML/JSON | AWS-only, deep AWS integration |

| CDK | Real languages, compiles to CloudFormation | Devs love it, AWS-only |

| Ansible | YAML (procedural) | Better for config management than provisioning |



\## What I want to try

\- Provision a small AWS setup (VPC + EC2 + S3) entirely in Terraform, using free-tier resources.

\- Move the state to a remote backend (S3 + DynamoDB lock).

\- Refactor the code into a reusable module.

\- Wire it into a GitHub Actions pipeline that runs `fmt`, `validate`, `plan` on PRs and `apply` on merge to `main`.

\- Try the same setup against \*\*LocalStack\*\* so I can practice without any AWS bill.



\## Sources

\- Terraform official docs (developer.hashicorp.com/terraform)

\- "Terraform: Up \& Running" — Yevgeniy Brikman

\- terraform-aws-modules on the public registry

\- The `tfsec` and `checkov` docs (for scanning your Terraform)

