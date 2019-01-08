---
layout: post
title:  "Controlling VMware NSX-T with Terraform"
date:   2019-01-07 22:20:56 +0100
tags:
    - Software Defined Networking
    - DevOps
permalink: nsx-terraform/
---
Hashicorp Terraform is a tool which enables you to safely and predictably create, change, and improve infrastructure by defining its configuration as code.

VMware NSX-T is a product which enables software defined network infrastructure.

The Terraform NSX-T provider allows us to deliver and maintain NSX-T configuration as code.

This is a walkthrough of how in a very few commands you can begin to control NSX-T configuration using NSX-T.

If starting from scratch [install a simple terraform server](https://learn.hashicorp.com/terraform/getting-started/install.html),  on CentOS / RHEL you would use these commands.

```bash
yum update -y
yum install wget net-tools unzip -y
cd /tmp
wget https://releases.hashicorp.com/terraform/0.11.11/terraform_0.11.11_linux_amd64.zip
mkdir /terraform
unzip terraform_0.11.11_linux_amd64.zip -d /terraform
cd /terraform
./terraform --version
```

Once we have this installed we can configure the  NSX-T Manager connection details as variables in a reusable variables file.

```bash
cat > /terraform/variables.tf << '__EOF__'
variable "nsx_manager" {}
variable "nsx_username" {}
variable "nsx_password" {}
__EOF__

cat > /terraform/terraform.tfvars << '__EOF__'
nsx_manager = "192.168.1.15"
nsx_username = "admin"
nsx_password = "VMware1!"
__EOF__
```

We can then create a very basic Terraform configuration file which uses these variables and then performs a simple action like creating a NSX IP Set.

```bash
cat > /terraform/nsx.tf << '__EOF__'
provider "nsxt" {
  host                  = "${var.nsx_manager}"
  username              = "${var.nsx_username}"
  password              = "${var.nsx_password}"
  allow_unverified_ssl  = true
  max_retries           = 10
  retry_min_delay       = 500
  retry_max_delay       = 5000
  retry_on_status_codes = [429]
}

resource "nsxt_ip_set" "ip_set1" {
  description  = "IS provisioned by Terraform"
  display_name = "IS"

  tag {
    scope = "color"
    tag   = "blue"
  }

  ip_addresses = ["1.1.1.1", "2.2.2.2"]
}
__EOF__
```

With this in place we can initialize Terraform and get it to pull down the NSX-T provider.

```bash
./terraform init
```

If all has gone well Terraform should initialize successfully.

<center><img src="/images/nsxt-terraform-init.png" width="50%"></center>

We can now look at the changes our Terraform configuration file will make to NSX.

```bash
./terraform plan
```

We should see the single resource defined in the Terraform configuration file.

<center><img src="/images/nsxt-terraform-plan.png" width="50%"></center>

Now we have verified it does what we hope we can then look to apply the change.

```bash
./terraform apply
```

Not as this is changing configuration we get asked to confirm action.

<center><img src="/images/nsxt-terraform-apply.png" width="50%"></center>

Once ran successfully we can then check in NSX GUI and confirm the IP Set is created.

<center><img src="/images/nsxt-terraform-confirm.png" width="50%"></center>