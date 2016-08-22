---
layout: post
title:  "Microsoft WSUS No Internet Connectivity"
date:   2014-04-04 16:59:56 +0100
tags:
    - Windows
permalink: microsoft-wsus-no-internet-connectivity/
---

On our team4tech.org project in rural Tanzania we are planning to use Windows 7 and 8 devices running 
MS Office.  While we plan to include all current patches within the image we’re deploying these we 
discussed ongoing supportability and patching. The challenge being that there is no internet connection 
within each school.

A possible solution to this would be to have have a computer will internet connectivity running the 
Windows Server Update Services. This would be configured to auto approve all critical and security 
updates for all versions of Windows and Office which are deployed. From this seed point we can then 
export the Update metadata using the command syntax like.

{% highlight cmd %}
"C:\Program Files\Update Services\Tools\wsusutil.exe" export "D:\WSUS_Meta\Metadata_YYYY_MM_DD.cab" 
"D:\WSUS_Meta\Metadata_YYYY_MM_DD.log"
{% endhighlight %}

We can then export the Approval metadata using the command syntax like.

{% highlight cmd %}
"C:\Program Files (x86)\Update Services 3.0 API Samples and 
Tools\WsusMigrate\WsusMigrationExport\WsusMigrationExport.exe" 
"D:\WSUS_Meta\Approvals_%Year%_%Month%_%Day%.xml"
{% endhighlight %}

As well as the cab and xml exports,  we must also copy the patch data out

{% highlight cmd %}
robocopy.exe \\wsus-server\d$\wsus\wsuscontent <usb drive letter>:\WSUS\WSUSContent /MIR /COPYALL
{% endhighlight %}

To import to the offline WSUS server within school is pretty much same as export in reverse,  so copy 
files back

{% highlight cmd %}
robocopy.exe <usb-letter>:\WSUS\WSUSContent \\wsus-server\d$\wsus\wsuscontent /MIR /COPYALL
{% endhighlight %}

Import Approval Metadata

{% highlight cmd %}
"C:\Program Files (x86)\Update Services 3.0 API Samples and Tools\WsusMigrate\WsusMigrationImport\WsusMigrationImport.exe" 
"D:\WSUS_Meta\Approvals_%YYYY%_%MM%_%DD%.xml" All DeleteUnmatchedTargetGroups
{% endhighlight %}

Import Update Metadata

{% highlight cmd %}
"C:\Program Files\Update Services\Tools\wsusutil.exe" import "D:\WSUS_Meta\Metadata_%YYYY%_%MM%_%DD%.cab" 
"D:\WSUS_Meta\Metadata_%YYYY%_%MM%_%DD%.log"
{% endhighlight %}

Your local offline WSUS server should now issue patches to clients. So you just need to ensure that all 
client devices point to the correct local URL as opposed to the update.microsoft.com default URL.