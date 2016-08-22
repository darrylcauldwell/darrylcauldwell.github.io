---
layout: post
title:  "Configuring Puppet In Homelab"
date:   2013-09-23 16:59:56 +0100
tags:
    - Homelab
permalink: puppet-learning-lab/
---

I'm a traditional wintel vmware engineer working in the UK in a team focusing on platform configuration automation 
for virtual and physical Windows hosted solutions. This video of how <a href="http://www.youtube.com/watch?v=UUgoiwEFe1A">
Puppet can be used to manage Amazon EC2</a>. Shortly after watching this VMware Hybrid Cloud Services launched and 
on watching <a href="http://www.youtube.com/watch?v=tp_1N3RSyUY">Nicholas Weavers video</a> from PuppetConf 2013. From 
this point on I decided that I needed to look into Puppet and how it could help me,  on initial looking it became 
clear that Puppet is traditionally *nix but does function on Windows so I thought I'd start here.

<H3>Environmental Summary</H3>
VMWare Workstation - 8.0.6 build-1035888  
Puppet Learning VM (1.5GB vRAM 1x vCPU 4.5GB HDD)<a href="http://info.puppetlabs.com/download-learning-puppet-VM.html">Download Here</a>  

<H3>Puppet Learning VM - Configuration</H3>
Once the Puppet Enterprise Learning VM is deployed from the OVF.  Its pretty intuitive setup I only changed to a 
bridged network connection for vNIC and installed VMware tools.

VMware tools install command syntax used
{% highlight bash %}
yum -y install perl
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cp /mnt/cdrom/VMwareTools-*.tar.gz /tmp
umount /mnt/cdrom
tar -zxf /tmp/VMwareTools-*.tar.gz -C /tmp
cd /
./tmp/vmware-tools-distrib/vmware-install.pl --default
rm -f /tmp/VMwareTools-*.tar.gz
rm -rf /tmp/vmware-tools-distrib
{% endhighlight %}

