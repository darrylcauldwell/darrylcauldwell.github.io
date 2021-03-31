+++
title = "Ansible For Windows Managed Nodes"
date = "2015-01-27"
description = "How To Setup An Ansible Test Lab For Windows Managed Nodes & Custom Windows Modules"
tags = [
    "ansible",
    "windows"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
On starting to look at learning using Ansible to manage Windows hosts, first step was to setup a test lab.

Setup an Ubuntu 14.04 or CentOS 7 and a Windows 2008 R2 virtual machines.

## Install Ansible (Ubuntu)
```bash
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
sudo pip install http://github.com/diyan/pywinrm/archive/master.zip#egg=pywinrm
```

## Install Ansible (CentOS 7 Minimal)
```bash
yum update
yum install net-tools
yum install epel-release
yum install ansible
```

## Configure WinRM On Windows 2008 R2 Guest
The configuration of Windows managed guests WinRM is automated and available in 
[this](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) powershell script.

## Create A Working Space Test Your Connectivity
Ansible is file based,  so here I create a folder underneath my home directory to store my lab configuration

```bash
mkdir -p ~/ansible_test/group_vars
```

We then create a file for holding hosts in lab and begin editing it

```bash
vi ~/ansible_test/host
```

Add the contents of this in the following format,  adding extra lines with extra IPs as needed

```yml
[windows]
<windows server ip address>
```

We then create a file for holding WinRM connectivity hosts in lab and begin editing it

```bash
vi /home/<user>/ansible_test/group_vars/windows.yml
```

Add the contents of this in the following format,  adding extra lines with username and password as needed

```bash
# it is suggested that these be encrypted with ansible-vault:
# ansible-vault edit group_vars/windows.yml
ansible_ssh_user: <admin user>
ansible_ssh_pass: <admin user password>
ansible_ssh_port: 5986
ansible_connection: winrm
```

Once we have these two files setup,  we can look to test connectivity

```bash
cd /home/dcauldwell/ansible_test
ansible windows -i host -m win_ping
```

Debugging is not enabled by default,  you might want to append -vvvvv to enable this if you have issue on first connect.

If you still have issues you can test connectivity using cURL

```bash
curl -vk -d "" -u "user:pass" https://<windows server ip address>:5986/wsman"
```

When you havethe win_ping module working,  you can look at running the other modules shipped with the core product a full list can be found here.  Maybe you might gather the Ansible facts using.

```bash
ansible <windows server ip address> -m setup
```

## Create A Custom Powershell Module
On looking at the core modules you might think your a bit limited but its easy to wrapper your existing Powershell logic as an Ansible module.

I decided to keep all my custom modules in the lab in there own folder and change the module path.

```bash
mkdir -p ~/ansible_test/group_vars/library
export ANSIBLE_LIBRARY=~/ansible_test/library/
```

An Ansible Powershell module is made up of two files,  one is the ps1 with the script contents and one is py which has description and examples.  The main difference appears to be to get the Powershell console output back to Ansible you need to form as object and convert that object to JSON.  It also appears not required but good practice to set an object to be returned with a changed flag to true or false unsure but believe this logic might be used at runtime to decide whether to call the handler.

An easy way to create a new module,  is to copy an existing one and rename it this way you get the supporting text and structure.

```bash
cp /usr/share/pyshared/ansible/modules/core/windows/win_ping* ~/ansible_test/library/
mv ~/ansible_test/library/win_ping.ps1 ~/ansible_test/library/<new module name>.ps1
mv ~/ansible_test/library/win_ping.py ~/ansible_test/library/<new module name>.py
```

A simple example use case might be if you wanted to call the Get-Host cmdlet to gather Powershell version your ps1 might read.

```powershell
#!powershell
# WANT_JSON
# POWERSHELL_COMMON
$data = Get-Host | Select Version
$result = New-Object psobject @{
get_host_version = $data
changed = $false
};
Exit-Json $result;
```

Using the same example your py might read

```python
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
```

Once we have these two module files,  we can look to test the new module

```bash
cd /home/dcauldwell/ansible_test
ansible windows -i host -m get_host
```

If its working you should get returned the output from Get-Host command

```bash
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
```

With this technique you should be able to form some simple modules.