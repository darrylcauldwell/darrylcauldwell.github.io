+++
title = "vRealize Orchestrator NSX Plug-in Troubleshooting"
date = "2018-02-06"
description = "vRealize Orchestrator NSX Plug-in Troubleshooting"
tags = [
    "nsx",
    "vro"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
The NSX-V Plug-in for vRealize Orchestrator offers some great functionally, however creating custom workflows caused me some headaches.

When building some custom workflows using the vRO API explorer there are inconsistencies, I've learned to not rely on the vRO API explorer for NSX plug-in documentation and instead use [www.vroapi.com](http://www.vroapi.com/Plugin/NSX/1.2.0) which is more complete.

Problems manifest themselves as failures to execute methods, but very little detail is returned with the error message. Within vRealize Orchestrator is a JavaScript runtime environment therefore we can install a plug-in such as [vCO-CLI](https://labs.vmware.com/flings/vco-cli) to get an exploratory programming interface and try the commands directly to see the return values. While this was useful it didn't help with all issues.

The NSX plug-in talks to NSX Manager via the REST API, so another avenue to follow is that you can enable HttpClient logging which captures the REST calls made by the Plug-in in real-time. After some searching [I found vRealize Orchestrator](https://www.vcoteam.info/articles/learn-vco/199-how-to-handle-vcenter-orchestrator-logs.html) uses [Apache log4j](https://logging.apache.org/log4j/2.x/).

We can edit the log4j configuration file

```bash
/etc/vco/app-server/log4j.xml
```

Within this configuration file we find section for the HttpClient org.apache.http

```xml
<category additivity="true" name="org.apache.http">
    <priority value="INFO"/>
</category>
```

We can enable HttpClient debugging by amending this to have value of DEBUG.

```xml
<category additivity="true" name="org.apache.http">
    <priority value="DEBUG"/>
</category>
```

Once the vRealize Orchestrator server services are restarted the logging level will change and events will be written to the following log file.

```bash
/var/log/vco/app-server/integration-server.log
```

Using a combination of these methods has enabled resolution of all issues experienced up to now. To note putting a vRealize Orchestrator server into debug mode will slow down the vRealize Orchetrator server considerably. It is recommended to use DEBUG mode temporarily.