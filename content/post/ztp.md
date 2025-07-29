+++
title = "Automating vSphere Provisioning from Bare Metal to Cluster"
date = "2025-07-09"
description = "Zero Touch ESXi Deployment with Kickstart and VCF Orchestrator: Automating Host Provisioning from Bare Metal to Cluster"
tags = [
    "vsphere",
    "esxi",
    "automation",
    "vcf",
    "kickstart"
]
categories = [
    "Automation",
    "VMware"
]
thumbnail = "clarity-icons/code-144.svg"
+++

Most remote sites don’t come with an IT expert, and noone enjoys clicking through ESXi setup screens. That’s where zero touch provisioning comes in. In this post, I’ll show you how to fully automate ESXi installation using a Kickstart script, use VCF Orchestrator to spin up a site specific cluster and onboard the hosts—no keyboard or mouse required.

But it’s not just about convenience. Automation brings consistency, reduces human error, and gives you a repeatable, auditable process—especially important if you’re working in regulated environments. Whether you’re scaling edge infrastructure or building out a new region, this approach ensures every host is deployed the same way, every time, without needing skilled hands on-site.

## HTTP Boot

When automating infrastructure provisioning, network booting is a key enabler—and there are a few options to choose from. PXE and iPXE have long been the go-to methods, but they rely on TFTP, which can be finicky to set up and troubleshoot, especially across subnets or in firewalled environments. UEFI HTTP Boot offers a simpler, more reliable alternative. It uses standard UEFI firmware to download bootloaders and installation files directly over HTTP, eliminating the need for a TFTP server entirely.

Most modern server vendors—including Dell, HPE, Cisco, and others—support UEFI HTTP Boot and provide ways to configure it remotely through their management interfaces such as iDRAC, iLO, and UCS Manager. While the specific setup steps and interface details vary by vendor and firmware version, UEFI HTTP Boot is quickly becoming the standard choice for zero touch provisioning, delivering consistent and reliable network boot capabilities across a wide range of hardware platforms.

### Setting Up A HTTP Server for ESXi UEFI HTTP Boot

To enable UEFI HTTP Boot, you need a web server to serve the ESXi installation files and Kickstart scripts. For this example, I built a simple Ubuntu server configured with a static IP using netplan. Once the Ubuntu server is up, install and start the Apache web server with these commands:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
```

After the service is running, you can verify it by opening your server’s IP address in a browser — you should see the default Apache welcome page.

Next, create directories to host the ESXi installer files and Kickstart configurations:

```bash
sudo mkdir -p /var/www/html/esxi9
sudo mkdir -p /var/www/html/kickstart
sudo chown -R www-data:www-data /var/www/html/esxi9 /var/www/html/kickstart
sudo chmod -R 755 /var/www/html/esxi9 /var/www/html/kickstart
```

With the folders ready, copy your ESXi 9 ISO to the Ubuntu server. To make the installation files accessible over HTTP, extract the ISO contents into the web directory. First, install the necessary tools to extract the ISO:

```bash
sudo apt install genisoimage p7zip-full -y
```

Then extract the ESXi installer:

```bash
sudo 7z x /home/ubuntu/VMware-VMvisor-Installer-9.0.0.24755229.x86_64.iso -o/var/www/html/esxi9
```

UEFI HTTP Boot expects a file named mboot.efi, but the ESXi ISO provides it as BOOTX64.EFI. To avoid boot failures, copy it to the expected filename:

```bash
sudo cp /var/www/html/esxi9/EFI/BOOT/BOOTX64.EFI /var/www/html/esxi9/mboot.efi
```

Your ESXi installer is now hosted and ready to be served over HTTP, laying the foundation for a fully automated, zero touch provisioning workflow.

### Host-Specific Provisioning with UEFI HTTP Boot

To make your ESXi deployment truly zero touch and environment-aware, it helps to tailor the provisioning flow for each individual host. A simple way to achieve this is by serving a unique boot.cfg file based on the server’s MAC address. This lets you assign static IPs, hostnames, and custom Kickstart scripts—without touching the console.

This relies on a naming convention used by many bootloaders (also common in PXE environments). The MAC address must be lowercase, use hyphens between octets, and be prefixed with 01-. The 01- prefix represents the hardware type code for Ethernet, as defined by the ARP protocol (where 01 = Ethernet). For example a MAC address like 00:11:22:33:44:55:66 becomes:
01-00-11-22-33-44-55-66

Create folder:
```bash
sudo mkdir -p /var/www/html/esxi9/01-<MAC address>
sudo chown -R www-data:www-data /var/www/html/esxi9/01-<MAC address>
sudo chmod -R 755 /var/www/html/esxi9/01-<MAC address>
```

Inside that folder, place a host-specific boot.cfg that defines the kernel, initrd, and Kickstart settings for that machine. Modify the boot.cfg file, make the following adjustments:

* Set a prefix: Add prefix=http://<your-server-ip>/esxi9
* Fix module paths: Remove all trailing slashes from kernel= and modules=
* Clean kernel options: Remove cdromBoot from the kernelopt= line

```bash
sudo vi /var/www/html/esxi9/EFI/01-<MAC address>/BOOT/BOOT.CFG
```

## Kickstart Script Distributed Version Control

To bring change control and transparency to my provisioning workflow, I’ve stored all my host-specific Kickstart files in a private GitHub repository. Instead of embedding the full script in the boot.cfg, I reference the raw URL of the Kickstart file, making it easy to audit, update, and roll back changes over time.

In each host’s boot.cfg, I point to the correct raw GitHub URL:

```bash
kernelopt=ks=https://raw.githubusercontent.com/<your-username>/<repo>/main/kickstart/ks-esxi-host01.cfg

## Example https://raw.githubusercontent.com/darrylcauldwell/kickstart/refs/heads/main/host102.ks
```

This approach lets me treat Kickstart files like code — complete with version control, pull requests, and commit history. It also opens the door for more structured configuration management across environments.  To support site-specific variations, I could use dedicated branches or even pin Kickstart URLs to specific commit hashes.

To avoid duplicating standard settings in every Kickstart file, I’m moving reusable config (like syslog, NTP, and DNS settings) into a separate script stored in the same GitHub repo. Then, in the %firstboot section of the Kickstart file, I use wget to pull that down during the install:

```bash
%firstboot --interpreter=busybox

# Fetch shared configuration script
wget -O /tmp/common-init.sh https://raw.githubusercontent.com/<your-username>/<repo>/main/scripts/common-init.sh
chmod +x /tmp/common-init.sh
/tmp/common-init.sh
```

This keeps each Kickstart file lightweight and focused on host-specific logic, while shared behavior remains centrally managed and versioned.

## UEFI HTTP Boot Discovery

UEFI HTTP Boot can obtain the URL of the EFI bootloader in two primary ways. First, many DHCP servers support specific DHCP options (such as option 67 or the newer HTTP Boot options defined in RFC 7830) that provide the exact HTTP URL for the EFI file. This allows the client to dynamically discover where to fetch its bootloader without manual configuration. Alternatively, you can specify the HTTP Boot URL directly in the UEFI firmware settings or through your server’s management interface (e.g., iDRAC, iLO, UCS Manager). This flexibility lets you tailor the boot process for different environments—whether centralized via DHCP or set individually per host—making zero touch provisioning both scalable and adaptable.

Instead of relying on DHCP to deliver the EFI boot file URL, I chose to configure the HTTP boot URL directly in the ESXi virtual machine’s UEFI firmware settings. This method gives precise control over the boot process on a per-host basis without requiring DHCP server configuration. It’s especially useful in nested ESXi lab environments or where DHCP option support is limited or unavailable. By specifying the exact URL in the firmware, each host knows exactly where to download its bootloader, streamlining the zero touch provisioning workflow. To set this up for a virtual ESXi host I used these steps:

1. Create an empty VM for UEFI HTTP Boot.
2. Open the VM’s Edit Settings dialog in the vSphere Client.
3. Scroll down and click VM Options > Advanced > Edit Configuration.
4. Add the following two advanced configuration entries:
    * Name: firmware.networkBootProtocol
      Value: httpv4
	* Name: firmware.networkBootUri
      Value: http://172.16.101.100/boot/esxi9/EFI/BOOT/BOOTX64.EFI

      (Replace with your actual HTTP server URL and path to the EFI boot file.)
5. Save and close the settings, then power on the VM.

With these settings, the VM’s UEFI firmware will directly request the specified EFI file over HTTP during boot, bypassing the need for DHCP-delivered boot URLs. This approach simplifies the boot configuration in virtualized environments and makes zero touch provisioning more predictable and manageable.

## VCF Orchestrator

VMware Cloud Foundation (VCF) Orchestrator is a powerful automation and orchestration engine designed to simplify and accelerate the deployment and lifecycle management of VMware infrastructure. It provides a set of APIs and workflows that enable administrators to programmatically manage hosts, clusters, and workload domains within a VCF environment.

In the following sections, I’ll explore how to integrate VCF Orchestrator with vCenter Server and use its workflow capabilities to automate complex tasks—such as onboarding newly provisioned ESXi hosts into clusters. This approach helps reduce manual steps, eliminate human error, improve consistency, and support compliance by maintaining a clear audit trail—all while enabling scalable operations across your infrastructure.

Each workflow task within VCF Orchestrator can generate detailed logs—including standard output messages, errors, and warnings—that are sent directly to the VCF Operations Logs. This centralized logging not only aids troubleshooting but also creates a comprehensive, auditable record of all automated actions performed during the provisioning and management process.

VCF Orchestrator can also integrate with external Configuration Management Databases (CMDBs) or similar systems of record to pull in site-specific variables and metadata. This dynamic data integration helps maintain accurate, up-to-date context for each site or environment, ensuring that orchestrated workflows execute with the correct configuration parameters tailored to the target location.

### Deploying VCF Orchestrator

Deploying VCF Orchestrator involves provisioning it as a dedicated appliance. Here’s a high-level outline of the process:

1. Download the OVA - Obtain the VCF Orchestrator appliance OVA from the VMware Customer Connect portal.
2. Deploy the OVA - Use the vSphere Client to deploy the OVA
3. Configure Networking, NTP and DNS
4. Initial Setup - Connect to CLI and configure identity provider, for simplicity I used vSphere SSO

### Adding vCenter Server as endpoint

By integrating with vCenter Server, Orchestrator workflows can leverage the full vSphere API. The integration allows Orchestrator to create resource objects and perform tasks. Additionally, all actions executed through the vCenter endpoint are logged and auditable, providing enhanced visibility and supporting compliance across your infrastructure.

For production environments, it’s best practice to create a dedicated service account with scoped permissions for integrating VCF Orchestrator with vCenter Server. This approach enhances security and auditability by limiting access to only the necessary resources and actions. However, for simplicity in my lab environment, I used the built-in administrator@vsphere.local account to streamline initial testing and workflow development.

To add  vCenter Endpoint expand the workflow library and run 'Add vCenter Endpoint'. The workflow will:

1. Prompt for connection details (hostname/IP, port, protocol)
2. Requires credentials
3. Stores the vCenter connection securely in Orchestrator’s endpoint list
4. Tests connectivity and verifies the endpoint before saving

### Orchestrator Workflow Import and Export

You can export a workflow from one environment and import it into another—such as from a lab to production—ensuring consistency and reducing manual effort. Exporting Orchestrator workflows as JSON and storing them in a GitHub repository brings significant benefits in terms of version control, collaboration, and portability. By treating your workflows as code, you can track changes over time, review diffs via pull requests, and maintain a clear history of updates—all of which support better governance and easier troubleshooting.

I have created an Orchestrator workflow which 

### Governance and Self-Service with VCF Automation

While VCF Orchestrator excels at automating infrastructure workflows, adding a governance layer ensures these powerful automations align with organizational policies and control requirements. VMware Cloud Foundation Automation (VCF Automation) provides this additional layer by enabling policy-driven management and approval workflows on top of Orchestrator’s capabilities.

With VCF Automation, users can submit self-service requests for infrastructure operations—such as provisioning or scaling hosts—while enforcing approval gates to maintain control and compliance. This means infrastructure changes are not only automated but also reviewed and authorized according to business rules, reducing risk.