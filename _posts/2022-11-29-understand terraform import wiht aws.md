---
title:  "Understand Terraform import with AWS IAM users and groups"
date:   2022-11-27 19:49:42 +0100
categories: 
  - Terraform
  - AWS

share: true
comments: true
classes: wide
author_profile: true
output: 
    html_document:
        fig_caption: yes
---



### How to import a resource created manually under Terraform management

To avoid configuration drift, Terraform has importing capabilities for most of the AWS resources. By importing a resource, Terraform stores in its state file the setup of the resource,

as it’s currently on the cloud. So, when you run `terraform plan`, the imported resources are compared with your configuration and the differences are presented.

One important thing to notice is that `terraform import` imports the resources in the state file, but doesn’t fill the resource definition on your .tf files. This is a process we have to perform manually,

**Usage:**

`terraform import [options] ADDRESS ID` where :

`ID`

is dependent on the resource type being imported. (you can find the id on the last line of every Terraform resource documentation)

`ADDRESS`

is a valid [resource address](https://www.terraform.io/cli/state/resource-addressing)

**Example importing Iam users created manually by the console under Terraform management**

suppose that we have an iam user **bob** created by the console, and we need to import it on our IaC code

add the resource block definition on your tf file :

```
resource "aws_iam_user" "existing_iam_user" {
  name = "bob"
}
```

and then Import it by running the following command


`terraform import aws_iam_user.existing_iam_user bob`


**Importing a group**

add the resource block definition on your tf file :

```
resource "aws_iam_group" "this" {

  name = "developers"

}
```

and then Import it by running the following command

`terraform import aws_iam_group.this developers`

you can do the same for every Terraform resource.