---
layout: post
title:  "Controlling NSX-T with Terraform with Jenkins"
date:   2019-01-08 22:20:56 +0100
tags:
    - DevOps
permalink: terraform-jenkins/
---

We would like to launch and destroy application specific NSX network infrastructure when we deploy our application. The Terraform plugin for Jenkins provides a build wrapper to perform this action.

If starting from scratch [install a simple jenkins server](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions),  on CentOS / RHEL you would use these commands.

```bash
yum update -y
yum install wget net-tools unzip -y
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins java -y
service jenkins start
chkconfig jenkins on
```

Once installed and running would load the [Terraform plugin](https://wiki.jenkins.io/display/JENKINS/Terraform+Plugin),  however this does not presently work as Terraform prompts for confirmation.  There is a [pending pull request to fix this](https://github.com/jenkinsci/terraform-plugin/pull/4/commits/47d6d3da54dd2cc437c1efb5df89cdccdb0f3eb0) until this gets merged we need a workaround.

We will be calling a shell script from within Jenkins we'll create a folder and give Jenkins user account permissions by running following.

```
mkdir /terraform
sudo usermod -a -G root jenkins
chmod -R g+w /terraform
```

We can then create a Jenkins job with Build contents which creates a Terraform file and applies this.  A simple example would be.

<center><img src="/images/nsxt-terraform-jenkins.png" width="50%"></center>

```
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

