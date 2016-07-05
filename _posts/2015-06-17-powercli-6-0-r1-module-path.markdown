---
layout: post
title:  "PowerCLI 6.0 R1 Module Path Issue"
date:   2015-06-17 16:59:56 +0100
tags:
    - Powershell
permalink: powercli-6-0-r1-module-path
---
The install for PowerCLI 6.0 R1 sets the module path for the logged on user but no one else,  
on a shared machine the other users get a module path error when PowerCLI launches and it tries 
to use a module to display version.

‘Get-PowerCLIVersion : The term ‘PowerCLIVersion’ is not recognised as the name of a cmdlet, 
function, script file, or operable program.’

<center><img src="/images/pcli6_r1_error.jpg" width="50%"></center>

The following code snippet from aaronk sets the correct values machine wide.

{% highlight cmd %}
$PowerCLIModulePath = "C:\Program Files (x86)\VMware\Infrastructure\vSphere PowerCLI\Modules"
$OldModulePath = [Environment]::GetEnvironmentVariable('PSModulePath','Machine')
if ($OldModulePath -notmatch "PowerCLI") {
Write-Host "[Adding PowerCLI Module directory to Machine PSModulePath]" -ForegroundColor Green
$OldModulePath += ";$PowerCLIModulePath"
[Environment]::SetEnvironmentVariable('PSModulePath',"$OldModulePath",'Machine')
} else {
Write-Host "[PowerCLI Module directory already in PSModulePath. No action taken]" -ForegroundColor Cyan
}
{% endhighlight %}
