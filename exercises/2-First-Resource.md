# Terraform Exercise 2 - First Resource

In this exercise we will create our first Terraform resource.

## A Server

Create a new file called `main.tf` and populate it with the contents:

```
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}

resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "t2.micro"

  tags = {
    Name = "HelloWorld"
  }
}
```

This configuration is designed to lookup an Ubuntu 14.04 AMI image, and create an EC2 instance
from it.

First, we need to initialize our Terraform workspace, by executing:

```terraform init```

Next, lets check that Terraform is going to create what we expect:

```terraform plan```

Inspect the output and connect it to the configuration above.

Now, lets instruct Terraform to create the resources:

```terraform apply```

This will take several minutes to complete, you can follow the log output in the console as Terraform
proceeds with building the infrastructure.

Once this has completed, you will now have a `terraform.tfstate` file in your current directory. Open
the file and inspect the contents, which you tie back to the resource configuration we used.

To illustrate the state management and idempotency of Terraform, again execute:

```terraform apply```

Note, that Terraform made absolutely no changes. This is because the current state already reflects
our declared configuration.

## Two Servers

Now we'll add a second EC2 instance, and iteratively apply our changes.

Open `main.tf` and alter the EC2 instance resource to the following:

```
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "t2.micro"

  count = 2

  tags = {
    Name = "HelloWorld"
  }
}
```

Note that we have added `count = 2`, instructing Terraform to create two of the specified resource.

If we now ask Terraform to output its change plan:

```terraform plan```

It will show that Terraform plans to add a single resource, which is our new instance. The existing
instance will remain unchanged.

Now apply the changes:

```terraform apply```

Terraform does not distinguish between create and update operations, you simply `apply` you current
configuration and Terraform will figure out the work to converge the current state to what has been
requested.

## Parameterizing

To create reusable configuration, its necessary to leverage parameterization. In Terraform this takes
the form of variables.

Lets parameterize the EC2 instance type, by changing the EC2 resource to:

```
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "${var.instance_type}"

  count = 2

  tags = {
    Name = "HelloWorld"
  }
}
```

Now create a separate file called `variables.tf`, and set the contents to:

```
variable "instance_type" {
    type = "string"
    default = "t2.small"
}
```

Terraform will automatically pick up all files with the extension `.tf` in the current directory, which
includes our variables file. There are no strict rules how content should be divided between these files,
and it is entirely up to you how to organize your Terraform configuration.

Execute:

```terraform plan```

The parameter we defined set the default value to `t2.small`, which is different to the existing `t2.micro`.
Hence, Terraform wants to rebuild our EC2 instances as type `t2.small`.

Apply the changes with:

```terraform apply```

We can now leverage this parameter by changing how we call Terraform. There are many ways to pass 
in parameter values that will override the default we defined, in this case we will use simple 
command line switches.

The following command will change our instance type back to `t2.micro`:

```terraform apply -var="instance_type=t2.micro"```

Lets apply this parameter value persistently by using a `tfvars` file. Create a file called `terraform.tfvars`
and populate it with the following:

```instance_type = "t2.micro"```

Now apply the configuration, removing the parameter we previously set, and Terraform will report 
no planned changes:

```terraform apply```

## Outputs

Finally, lets add an output. We're going to output the EC2 instance IDs and use these later.

Create a file called outputs.tf and populate it with the following:

```
output "instance_ids" {
  value = "${aws_instance.web.*.id}"
}
```

You'll notice the output from the Terraform CLI now includes and `Outputs` section, which specifies 
the EC2 instance IDs that our Terraform has created. This happens when you specify outputs in the 
"root" module.