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
  description  = "IP Set provisioned by Terraform"
  display_name = "IP Set"

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

If we would like to launch and destroy application specific NSX network infrastructure when we deploy our application. We would do this with a CI/CD pipeline tool like Jenkins will walk through this. If starting from scratch [install a simple jenkins server](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions), on CentOS / RHEL you would use these commands.

```bash
yum update -y
yum install wget net-tools unzip -y
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins java -y
service jenkins start
chkconfig jenkins on
```

Once installed and running we would normally load the [Terraform plugin](https://wiki.jenkins.io/display/JENKINS/Terraform+Plugin),  however this does not presently work as Terraform prompts for confirmation.  There is a [pending pull request to fix this](https://github.com/jenkinsci/terraform-plugin/pull/4/commits/47d6d3da54dd2cc437c1efb5df89cdccdb0f3eb0) until this gets merged we need a workaround.

To workaround this issue we can still control Terraform with Jenkins by calling a shell script from within Jenkins job.  To do this we will create a folder and give Jenkins user account permissions by running following.

```bash
mkdir /terraform
sudo usermod -a -G root jenkins
chmod -R g+w /terraform
```

We can then create a Jenkins job with Build contents which creates a Terraform file and applies this.  A simple example would be.

<center><img src="/images/nsxt-terraform-jenkins.png" width="50%"></center>

```bash
cd /terraform

cat > /terraform/nsx.tf << '__EOF__'
provider "nsxt" {
  host                  = "192.168.1.15"
  username              = "admin"
  password              = "VMware1!"
  allow_unverified_ssl  = true
  max_retries           = 10
  retry_min_delay       = 500
  retry_max_delay       = 5000
  retry_on_status_codes = [429]
}

resource "nsxt_ip_set" "ip_set1" {
  description  = "IP Set provisioned by Terraform"
  display_name = "IP Set"

  tag {
    scope = "color"
    tag   = "blue"
  }

  ip_addresses = ["1.1.1.1", "2.2.2.2"]
}
__EOF__

./terraform init

./terraform apply -auto-approve

rm -f nsx.tf
```

This is an overly simple example and more likely we would pull a config file from distributed source control such as github.