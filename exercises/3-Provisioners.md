# Terraform Exercise 2 - First Resource

In this exercise we will try some Terraform provisioners

## Preparing our EC2 Instance

First, we have a few configuration changes to make in order to open up some access to our EC2 instance:

- Create an EC2 keypair, that provides us with credentials to SSH in to the instance
- Create a security group that opens up ports 22 and 80

We're also going to generate a random string to append to our AWS resource names, so we don't get
naming conflicts in the same account.

Add the following to `main.tf`:

```
resource "random_string" "generated_id" {
  length = 6
  special = false
}

resource "tls_private_key" "generated_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "generated_key" {
  key_name   = "workshop-keypair-${random_string.generated_id.result}"
  public_key = "${tls_private_key.generated_key.public_key_openssh}"
}

resource "aws_security_group" "security_group" {
  name        = "workshop-${random_string.generated_id.result}-sg"
  description = "workshop-${random_string.generated_id.result}-sg"

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    protocol    = "tcp"
    from_port   = 22
    to_port     = 22
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    protocol    = "tcp"
    from_port   = 80
    to_port     = 80
  }

  egress {
    cidr_blocks = ["0.0.0.0/0"]
    protocol    = "-1"
    from_port   = 0
    to_port     = 0
  }
}
```

And alter our `aws_instance` resource to reference this new configuration:

```
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "${var.instance_type}"

  count = 2

  tags = {
    Name = "HelloWorld"
  }

  key_name      = "${aws_key_pair.generated_key.key_name}"

  vpc_security_group_ids = ["${aws_security_group.security_group.id}"]
}
```

Before we `apply` our changes we're going to have to re-initialize our Terraform directory,
as we've imported a resource that requires a new provider (`random_string`):

```terraform init```

Now `apply` our changes before we go any further:

```terraform apply```

## Installing Nginx

Open up the `main.tf` file and change the `aws_instance` resource to the following:

```
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "${var.instance_type}"

  count = 2

  tags = {
    Name = "HelloWorld"
  }

  key_name      = "${aws_key_pair.generated_key.key_name}"

  vpc_security_group_ids = ["${aws_security_group.security_group.id}"]

  provisioner "remote-exec" {
    inline = ["sudo apt-get update && sudo apt install nginx"]
  }

  connection {
    user        = "ubuntu"
    private_key = "${tls_private_key.generated_key.private_key_pem}"
  }
}
```

Now, when the EC2 instances come up Terraform will initiate an SSH connection and run the 
provided scripts, while install Nginx server.

So we can test that Nginx is working, add an additional output of the EC2 instances public 
IP addresses. Add the following to the `outputs.tf` file:

```
output "public_ip" {
  value = "${aws_instance.web.*.public_ip}"
}
```

Finally, we can apply these changes and let Terraform work its magic:

```terraform apply```

After the `apply` is complete, you should see an additional section in the output:

```
public_ip = [
    54.153.84.136,
    54.67.76.184
]
```

Copy one of these IP addresses and access it using your browser. You should see the Nginx welcome page.
