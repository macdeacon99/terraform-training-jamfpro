# 🧱 Module 01 - Terraform Basics

This module will introduce the basics of Terraform and working with Configuration as Code.

This module presumes you know the following:

- You understand what Configuration as Code is.
- You have an understanding of what Terraform as a concept is.

## 🎯 Learning Objectives

By the end of this module, participants will:

- Understand what **providers**, **resources**, and the **state file** are in Terraform.
- Be able to identify the structure of a resource block and its key components.
- Use the Terraform commands `plan`, `apply`, and `destroy` to manage infrastructure.
- Understand where to look for documentation

---

## 🧩 1. Introduction to Providers

### What is a Provider?

A **provider** is a plugin that lets Terraform talk to an external system — such as AWS, Azure, Jamf Pro, or GitHub.

Each provider knows how to:

- Authenticate with that system
- Read, create, update, and delete resources
- Map Terraform configuration to real-world APIs

Think of a provider as a **translator** between Terraform and your environment.

### Example

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

This tells Terraform:

> "Use the AWS provider, and work in the us-east-1 region."

For Jamf Pro, it might look like:

```hcl
terraform {
  required_providers {
    jamfpro = {
      source  = "deploymenttheory/jamfpro"
      version = "0.26.0"
    }
  }
}

provider "jamfpro" {
    url = "https://yourcompany.jamfcloud.com"
    username = var.jamf_username
    password = var.jamf_password
}
```

### Key Concept

- You can use multiple providers in a single Terraform configuration.
- Providers are versioned — you can lock them to specific versions for consistency.

### 🧠 Exercise (Walkthrough)

1. In your working directory, create a file called provider.tf.
2. Add the Jamf provider block.
3. Configure the provider with variables required.
   - You may want to look at the documentation to see what is required.
4. Run `terraform init` — this downloads the provider plugin.

## 📦 Introduction to the Terraform State File

### What is the State File?

Terraform keeps track of everything it manages in a file called terraform.tfstate.

This file records:

- What resources exist
- Their configuration
- Relationships between them

Think of it like Jamf’s inventory database — it tells Terraform what’s already there.

### Why It Matters

The state file allows Terraform to know what needs to be changed.

It prevents Terraform from recreating resources unnecessarily.

It enables drift detection — spotting changes made outside Terraform.

### ⚠️ Important Notes

Never edit the state file manually.

Don’t share it casually — it can contain sensitive info (like credentials).

For teams, use remote state storage (e.g., S3, Terraform Cloud).

### 🧠 Exercise (Walkthrough)

1. Run terraform apply on a simple resource.
2. Open terraform.tfstate and inspect what’s inside (read-only!).
3. Notice how Terraform records the resource ID and attributes.
