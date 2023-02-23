---
title:  "Managing Multiple Environments using Terraform Workspace vs Folders vs Terragrunt"
date:   2023-02-23 17:20:00 +0100
categories: 
  - Terraform

share: true
comments: true
classes: wide
author_profile: true
output: 
    html_document:
        fig_caption: yes
---

As your infrastructure grows, managing multiple environments (such as development, staging, and production) can become challenging. In this blog post, we'll explore three popular approaches in Terraform open source version to managing multiple environments: Terraform workspace, folder structure, and Terragrunt.

**Terraform Workspace**

Terraform workspace is a built-in feature of Terraform that allows you to manage multiple environments within a single Terraform configuration. Workspaces enable you to create separate state files for each environment, keeping the configuration separate and distinct. This means that you can make changes to one environment without affecting the others.

To create a workspace, you can use the terraform workspace new command. For example, if you want to create a workspace for the development environment, you can run:

`terraform workspace new dev`

This creates a new workspace called dev. You can then switch to this workspace using the terraform workspace select command:

`terraform workspace select dev`

Once you have switched to the workspace, any changes you make to your Terraform configuration will be applied to that workspace's state file.

Using Terraform workspace simplifies the management of multiple environments as it allows you to use a single Terraform configuration file for all environments. However, it can become complex if you have many workspaces with many resources.

**Folder Structure**
Another way to manage multiple environments is to use a folder structure. This involves organizing your Terraform configuration files into separate folders, one for each environment. Each folder contains a complete set of Terraform configuration files for that environment.

For example, you could have the following folder structure:
```
.
├── dev
│ ├── main.tf
│ ├── variables.tf
│ └── ...
├── staging
│ ├── main.tf
│ ├── variables.tf
│ └── ...
└── prod
├── main.tf
├── variables.tf
└── ...
```
Using a folder structure provides a clear separation between environments, making it easier to manage changes and apply them to specific environments. However,as the size of the infrastructure grows, having to maintain all of this duplicated code between environments becomes more error prone as all environments will likely contain the same code with a different sizing.

If you want to have a look at a real world example using folder structure approach see https://github.com/antonbabenko/terraform-best-practices/tree/master/examples

**Terragrunt**

[Terragrunt](https://github.com/gruntwork-io/terragrunt) is a tool that provides extra functionality on top of Terraform. It simplifies infrastructure management by providing a layer of abstraction that enables you to manage multiple Terraform configurations and workspaces in a more streamlined way.

Terragrunt uses a configuration file called terragrunt.hcl to define the Terraform configurations and workspaces that it should manage. This allows you to create a modular, reusable infrastructure codebase that can be easily shared and reused across projects.

One of the key features of Terragrunt is its ability to automatically generate Terraform configuration files based on templates. This allows you to create a single template that can be used across multiple environments, with Terragrunt automatically updating the relevant variables and settings based on the current workspace.

Terragrunt also provides features like automatic state locking and remote state management, simplifying the management of Terraform state files across multiple environments.

Real world example using Terragrunt : https://github.com/gruntwork-io/terragrunt-infrastructure-live-example

Using Terragrunt may require some learning, but it is definitely worth it.

**Conclusion**

Managing multiple environments can be challenging, picking up an approach can be difficult but in general the most used on is *Folder structure* as it easy to manage and don't require learning a new tool,