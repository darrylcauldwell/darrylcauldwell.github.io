---
layout: post
title:  "How To Setup An Ansible Test Lab For Windows Managed Nodes & Custom Windows Modules"
date:   2015-01-27 16:59:56 +0100
tags:
    - Ansible
permalink: how-to-setup-an-ansible-test-lab-for-windows-managed-nodes-custom-windows-modules
---

On starting to look at learning using Ansible to manage Windows hosts, first step was to setup a test lab.

Setup an Ubuntu 14.04 or Cento7 and a Windows 2008 R2 virtual machines.

<H3>Install Ansible (Ubuntu)</H3>
{% highlight bash %}
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
sudo pip install http://github.com/diyan/pywinrm/archive/master.zip#egg=pywinrm
{% endhighlight %}


<H3>Install Ansible (Centos7 Minimal)</H3>
{% highlight bash %}
yum update
yum install net-tools
yum install epel-release
yum install ansible
{% endhighlight %}

<H3>Configure WinRM On Windows 2008 R2 Guest</H3>
The configuration of Windows managed guests WinRM is automated and available in 
[this](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) 
powershell script.

<H3>Create A Working Space Test Your Connectivity</H3>
Ansible is file based,  so here I create a folder underneath my home directory to store my lab configuration

{% highlight bash %}
mkdir -p /home/<user>/ansible_test/group_vars
{% endhighlight %}

We then create a file for holding hosts in lab and begin editing it

{% highlight bash %}
vi /home/<user>/ansible_test/host
{% endhighlight %}

Add the contents of this in the following format,  adding extra lines with extra IPs as needed

{% highlight python %}
[windows]
<windows server ip address>
{% endhighlight %}

We then create a file for holding WinRM connectivity hosts in lab and begin editing it

{% highlight bash %}
vi /home/<user>/ansible_test/group_vars/windows.yml
{% endhighlight %}

Add the contents of this in the following format,  adding extra lines with username and password as needed

{% highlight bash %}
# it is suggested that these be encrypted with ansible-vault:
# ansible-vault edit group_vars/windows.yml
ansible_ssh_user: <admin user>
ansible_ssh_pass: <admin user password>
ansible_ssh_port: 5986
ansible_connection: winrm
{% endhighlight %}

Once we have these two files setup,  we can look to test connectivity

{% highlight bash %}
cd /home/dcauldwell/ansible_test
ansible windows -i host -m win_ping
{% endhighlight %}

Debugging is not enabled by default,  you might want to append -vvvvv to enable this if you have issue on first connect.

If you still have issues you can test connectivity using cURL

{% highlight bash %}
curl -vk -d "" -u "user:pass" https://<windows server ip address>:5986/wsman"
{% endhighlight %}

When you havethe win_ping module working,  you can look at running the other modules shipped with the core product a full list can be found here.  Maybe you might gather the Ansible facts using.

{% highlight bash %}
ansible <windows server ip address> -m setup
{% endhighlight %}

<H3>Create A Custom Powershell Module</H3>
On looking at the core modules you might think your a bit limited but its easy to wrapper your existing Powershell logic as an Ansible module.

I decided to keep all my custom modules in the lab in there own folder and change the module path.

{% highlight bash %}
mkdir -p /home/<user>/ansible_test/group_vars/library
export ANSIBLE_LIBRARY=/home/<user>/ansible_test/library/
{% endhighlight %}

An Ansible Powershell module is made up of two files,  one is the ps1 with the script contents and one is py which has description and examples.  The main difference appears to be to get the Powershell console output back to Ansible you need to form as object and convert that object to JSON.  It also appears not required but good practice to set an object to be returned with a changed flag to true or false unsure but believe this logic might be used at runtime to decide whether to call the handler.

An easy way to create a new module,  is to copy an existing one and rename it this way you get the supporting text and structure.

{% highlight bash %}
cp /usr/share/pyshared/ansible/modules/core/windows/win_ping* ~/ansible_test/library/
mv ~/ansible_test/library/win_ping.ps1 ~/ansible_test/library/<new module name>.ps1
mv ~/ansible_test/library/win_ping.py ~/ansible_test/library/<new module name>.py
{% endhighlight %}

A simple example use case might be if you wanted to call the Get-Host cmdlet to gather Powershell version your ps1 might read.

{% highlight bash %}
#!powershell
# WANT_JSON
# POWERSHELL_COMMON
$data = Get-Host | Select Version
$result = New-Object psobject @{
get_host_version = $data
changed = $false
};
Exit-Json $result;
{% endhighlight %}

Using the same example your py might read

{% highlight bash %}
DOCUMENTATION = '''
---
module: get_host
version_added: "0.1"
short_description: Call Get-Host cmdlet.
description:
- Call Get-Host cmdlet
'''
EXAMPLES = '''
# Test connectivity to a windows host
ansible winserver -m get_host
# Example from an Ansible Playbook
- action: get_host
'''
{% endhighlight %}

Once we have these two module files,  we can look to test the new module

{% highlight bash %}
cd /home/dcauldwell/ansible_test
ansible windows -i host -m get_host
{% endhighlight %}

If its working you should get returned the output from Get-Host command

{% highlight bash %}
dcauldwell@ansible-server:~/ansible_test$ ansible windows -i host -m get_host
192.128.0.60 | success >> {
"changed": false,
"get_host_version": {
"Version": {
"Build": -1,
"Major": 4,
"MajorRevision": -1,
"Minor": 0,
"MinorRevision": -1,
"Revision": -1
}
}
{% endhighlight %}

With this technique you should be able to form some simple modules.