---
layout: post
title:  "Image Builder: Create A Custom ESXi ISO"
date:   2014-02-26 16:59:56 +0100
tags:
    - Powershell
permalink: image-builder/
---
If you find an occasion where some drivers you need are missing from the VMware provided ISO for installing
ESXi you can use Image Builder to create your own custom ISO file.

<h3>ESXi Image Builder Requirements</h3>
<ul>
	<li>    Microsoft .NET 2.0</li>
	<li>    Microsoft PowerShell</li>
	<li>    vSphere PowerCLI</li>
</ul>

<h3>Process Overview</h3>

The flow of what you are trying to create

<center><img src="/images/image-builder1.jpg" width="50%"></center>

A short video of adding FusionIO device drivers to ESXi 5 and forming out to iso

[youtube=https://www.youtube.com/watch?v=4r0bw7k3Hx8]

<h3>Install/Uninstall Custom Drivers</h3>
{% highlight cmd %}
# Import VMware and VMware Image Builder SnapIn's
Add-PSSnapin VMware.VimAutomation.Core
Add-PSSnapin VMware.ImageBuilder
# Download the offline bundle version of ESXi from vmware.com
# import the VMware image into image builder builder
Add-EsxSoftwareDepot "C:VMsHome Lab SetupCustomDepotVMware-ESXi-5.0.0-469512-depot.zip"
# Download the offline driver bundle you want to add from vendor example HP IO Accelerator
# and import the VMware image into image builder builder
Add-EsxSoftwareDepot "C:VMsHome Lab SetupCustomDepotioSphere3.6.1_ESXi-5.xUtilitiesfusionio-cimprovider-esxi5-bundle-3.6.1-114.zip"
# View contents of depot
Get-EsxSoftwareDepot
# View all images in depot
Get-EsxImageProfile
# Clone an existing image to form a new custom image
New-EsxImageProfile -CloneProfile ESXi-5.0.0-469512-standard -Name ESXi-5.0.0-469512.FusionIO -AcceptanceLevel PartnerSupported -Vendor HP
# Find name of packages
Get-EsxSoftwarePackage | where {$_.Vendor -eq "Fusion-io"} | Format-Table -AutoSize
# Add additional packages into new custom image
Add-EsxSoftwarePackage -ImageProfile ESXi-5.0.0-469512.FusionIO -SoftwarePackage fio
# Export image as ISO
Export-EsxImageProfile -ImageProfile ESXi-5.0.0-469512.FusionIO -ExportToIso -FilePath "C:VMsHome Lab SetupCustomDepotESXi-5.0.0-469512.FusionIO.iso"
{% endhighlight %}
