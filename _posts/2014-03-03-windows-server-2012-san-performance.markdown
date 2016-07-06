---
layout: post
title:  "Windows Server 2012 - SAN Performance"
date:   2014-03-03 16:59:56 +0100
tags:
    - Windows
    - Storage
permalink: windows-server-2012-san-performance
---
In a newly deployed Windows Server 2012 HyperV environment with around sixty nodes, running a 
couple of hundred Windows Server 2012 virtual machines running various applications.  We found t
he EMC VMX storage which underpinned this became massively busy with Write IO at around 3am,  
the Write IO was mostly coming from TRIM and Unmap commands,  the amount has a dramatic impact 
and leads to lots of 153 errors on the server estate,  after one to two hours the Write IO subsided. 
I wrote a script to audit the estate to identify if there was any pattern to the errors. We found 
the errors correlated to the 3am Window also.

EMC initially said the cause of this was the way the VNX was interpreting ODX command and they 
asked if we disable ODX on all HyperV servers. I used a PowerShell command to extract out all 
servers in AD with a ServiceConnection point of HyperV,  then looked to audit ODX on these and 
then disable.

Since disabling the problem persisted second thoughts were that there was some tasks scheduled 
within the applications running on the VMs which was triggering this event,  after speaking 
to the product groups they said nothing was scheduled at this time. We continued our search 
within the HyperV communities and found 
[this article](http://larsjoergensen.net/windows/windows-server/windows-server-2012/server-2012-automatic-maintenance-killed-san) 
which suggested some scheduled tasks are now enabled by default within Windows Server 2012.  On 
looking there appear two 

{% highlight cmd %}
\Microsoft\Windows\TaskScheduler\Regular Maintenance
{% endhighlight %}
which has a Trigger Daily At 03:00 (if 2012 R2 this would be 2am as the default time changed) and 

{% highlight cmd %}
\Microsoft\Windows\TaskScheduler\Maintenance Configurator
{% endhighlight %}

which has a Trigger Daily At 01:00. 

The Action for these tasks is “Custom Handler”, Microsoft publish 
no details on what these tasks do. We logged a call with Microsoft and they also did not know what these, 
but they did say that tasks with a “Custom Handler” are umbrella’s for other tasks.

As Regular Maintenance appeared to be the most likely due to execution time, I viewed its Last 
Run Time.

<center><img src="/images/tshed.png" width="50%"></center>

I then manually worked through each Windows task in Task Scheduler and took note of which Tasks 
had Last Run Time which matched that of “Regular Maintenance”.

{% highlight cmd %}
\Microsoft\Windows\.NET Framework\.Net Framework NGEN v4.0.30319
\Microsoft\Windows\.NET Framework.Net Framework NGEN v4.0.30319 64
\Microsoft\Windows\Application Experience\AitAgent
\Microsoft\Windows\Application Experience\ProgramDataUpdater
\Microsoft\Windows\ApplicationData\CleanupTermporaryState
\Microsoft\Windows\Chkdsk\ProactiveScan
\Microsoft\Windows\Customer Experience Improvement Program\KernelCeipTask
\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip
\Microsoft\Windows\Defrag\ScheduledDefrag
\Microsoft\Windows\Device Setup\Metadata Refresh
\Microsoft\Windows\PI\Sqm-Tasks
\Microsoft\Windows\Registry\RegIdleBackup
\Microsoft\Windows\Servicing\StartComponentCleanup
\Microsoft\Windows\Shell\CreateObjectTask
\Microsoft\Windows\Time Synchronization\SynchronizeTime
{% endhighlight %}

Of these tasks,  we identified *ScheduleDefrag* and *Chkdsk* as potentially moving a lot of data around 
and freeing up blocks which would trigger Windows to isssue TRIM \ Unmap commands for the SAN to 
\recover the space within the Thin LUN.

While looking at Trim \ Unmap on 2012 I  found that you can disable Trim \ Unmap commands being issued 
by setting the [DisableDeleteNotify](http://technet.microsoft.com/en-us/library/cc785435.aspx) file system 
attribute.

As there are around 350 virtual servers per site, it was not practical to audit, enable and disable each of 
the four configuration items, *ODX*, *Trim on delete*, *ScheduledDefrag* and *ChkDskProactiveScan* so I 
wrote a modular PowerShell script to get current state,  enable and disable each against a list of 
servers,  this script can be [found here.](https://github.com/darrylcauldwell/WindowsTrimAndUnmap)

Our storage vendor provided this excellent detail 
[EMC KB182688](https://emc--c.na5.visual.force.com/apex/KB_BreakFix_1?id=kA1700000000tup).

I would like to take functionality of thin provisioning so we have left enabled TrimOnDelete, but I could 
not see why you would want to Defrag a volume sitting on a Thin SAN LUN so we have disabled the ScheduledDefrag 
task. By disabling the task prevents Regular Maintenance running this at the 3am Window.*

For the medium to longer term we are looking to prove the performance differential between 2012 R0 
to R2 and if beneficial look to migrate to that.