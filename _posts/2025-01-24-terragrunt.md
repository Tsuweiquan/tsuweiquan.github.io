---
layout: post
title: Terragrunt
date: 2025-01-24 00:00:00
description: Setting up IaC with Terragrunt
tags: Terragrunt AWS Terraform
categories: infrastructure-as-code
---

# Pre-requisite: Terraform
Before understanding more about Terragrunt, it is required to know what is terraform and how can Terragrunt solve some issues in Terraform.

Terraform, an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share. 
- Ref: https://developer.hashicorp.com/terraform/intro

Terraform allow us to create infrastructure or cloud resources by writing a code of resource block. Undeniably, it's a great tool when we need to create the same infrastructure across different environment. We just need to apply the same configuration or code block across different environment, and we should expect the configuration of the infrastructure to be equal across different environments.

A typical yet simplest terraform folder structure will look like the following, where each environment would have their own state file (terraform.tfstate). Resources meant to be created in that environment should be written to its respective folder. 

We can run terraform in the `terraform_root/environments/dev/` folder to deploy/create resources for `dev` environment, similarly for `staging` and `prod`.
```
.
└── terraform_root/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── versions.tf
    ├── provider.tf
    ├── README.md
    └── environments/
        ├── dev/
        │   ├── main.tf
        │   └── terraform.tfvars
        ├── staging/
        │   ├── main.tf
        │   └── terraform.tfvars
        └── prod/
            ├── main.tf
            └── terraform.tfvars
```

### Terraform Impending Problems

- Code Duplication
  - Resources created in `dev` environment needs to be copied over to `staging` and `prod` environment. Copying of resources might look alright but when your environment scales up, this become really taxing and error-prone.
  - Not a DRY (Don't Repeat Yourself) approach.
  - Very prone to Human Error
- Single state per environment can be hard to manage
  - Terraform state can easily be bloated up.
  - `Terraform plan/apply/refresh` will take longer and longer as your infrastructure scales up
- Difficult to do logical separation of projects
  - Tagging could be a hassle
- Complexity increases with Region based deployments
  - If you are using AWS, how can we tweak the folder structure to accommodate deployment to different regions (`us-east-1` & `ap-southeast-1`)? 
    - The simplest way is to rename `dev` folder to `dev-us` and create another `dev-sg` folder. Next, copy the required code in `dev-us` into `dev-sg` and start running terraform in `dev-sg` to deploy resources in `ap-southeast-1` region.
      - This method is straight-forward but just not ideal...

These are the problems I faced when I was working in my organization. Terraform state becomes so huge that we start to use `-target=` arguments during `terraform plan` to speed up the deployment process. 
With much faster processing speed and efficiency, everyone starts to use `-target=` as the default practice, which is a BAD PRACTICE!

> Targeting individual resources can be useful for troubleshooting errors, but should not be part of your normal workflow.
  - ref: [Target resources](https://developer.hashicorp.com/terraform/tutorials/state/resource-targeting) 

As time goes by, with every run becomes a target run, this results in *Infrastructure Drift*. God knows when was the last time we ran terraform apply on the entire plan, and this becomes a big problem! No engineers would take the risk to run a full sync, risking the modification or deletion of some resources that we have no idea of.

Terragrunt can absolutely solve the issues I mentioned above!
<hr>

# Terragrunt


