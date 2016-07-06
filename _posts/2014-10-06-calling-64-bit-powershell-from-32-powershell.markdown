---
layout: post
title:  "Calling 64bit Powershell From 32bit Powershell"
date:   2014-10-06 16:59:56 +0100
tags:
    - Powershell
permalink: calling-64-bit-powershell-from-32-powershell
---
The %windir%\System32 directory is reserved for 64-bit applications. Most file names were not changed when 64-bit versions of the DLLs were created, so 32-bit versions are stored in a different directory. WOW64 hides this difference by using a <em>file system redirector</em>. As such whenever a 32-bit application attempts to access %windir%\System32, the access is redirected to %windir%\SysWOW64.

If your using 2008 or newer fear not as there is a method to path from 32 bit running program to the 64 bit path using a little known system redirector C:\Windows\Sysnative.  If your still using 2003 then you can apply <a href="http://support.microsoft.com/kb/942589" target="_blank">this patch</a> to get the Sysnative redirector.

Using this new knowledge we can then form any command we need to run in 64 bit powershell as a PS1 and invoke it using the 64 bit powershell.exe.

{% highlight cmd %}
$ScriptText = 'Install-WindowsFeature Web-Server'
$ScriptText | Set-Content 'Script.ps1'
C:\Windows\Sysnative\Powershell.exe ".\Script.ps1"
{% endhighlight %}