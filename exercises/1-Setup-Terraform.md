# Terraform Exercise 1 - Setup Terraform

In this exercise we will setup Terraform to work with AWS.

## Download Terraform

If on OSX you can use Homebrew:

```brew install terraform```

Otherwise, you can download the latest version of Terraform here:

```https://www.terraform.io/downloads.html```

Ensure Terraform is placed on your PATH so that the `terraform` command works from anywhere.

## Configure AWS

The workshop leader will provide you with a `terraform-env` file, which will automatically
set up the relevant environment variables for your assigned AWS login.

- Place the `terraform-env` file in a local directory
- From a Linux/Cygwin command-line execute `source terraform-env`