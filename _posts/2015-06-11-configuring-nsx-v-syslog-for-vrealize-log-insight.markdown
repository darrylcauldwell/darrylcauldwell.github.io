---
layout: post
title:  "Configuring NSX-V Syslog For vRealize Log Insight"
date:   2015-06-11 16:59:56 +0100
tags:
    - VMware
    - Software Defined Networking
permalink: configuring-nsx-v-syslog-for-vrealize-log-insight
---
I find myself with the need to configure Log Insight to manage an NSX-V installation. With how simple configuring 
the vSphere components with Log Insight it was a surprise to find the rather less simple steps needed for NSX-V.

There are three items of NSX-V which we want to capture syslogs from
<ul>
 	<li>NSX Manager</li>
 	<li>NSX Controllers</li>
 	<li>NSX Edges</li>
</ul>

Rather strangely as they part of the same product suite each has a very different method for configuring.

<H3>NSX Manager</H3>

To configure NSX Manager syslog server connect to administration web console and select 'Manage Appliance 
Settings'.

<center><img src="/images/NSX_Mgr_Menu.jpg" width="50%"></center>

Ensure General is selected in the left pane,  then Edit 'Syslog Server' and enter IP address or FQDN of the 
Log Insight server on Port '514' and protocol UDP.

<center><img src="/images/NSX_Mgr_Cfg.jpg" width="50%"></center>

<H3>NSX Controllers</H3>
To configure NSX controllers syslog server you need to use a REST client,  the following steps are done using 
Mozilla Firefox add-in 'RESTclient', but will be similar for all clients.

Within RESTclient click Authentication and select Basic Authentication,  you will be prompted for username 
and password.  This will be used by the client to connect to the controller as such enter credentials which 
have access to NSX Controllers.

<center><img src="/images/RESTclient_Auth.jpg" width="50%"></center>

The NSX Controller API we will be using takes data in XML format.  To ensure data is processed correctly 
we add a Header stating our content type to our requests. Within RESTclient click '*Header*' menu and click 
'*Custom Header*' for the header give name '*Content-Type*' and value '*application/xml*'

<center><img src="/images/RESTclient_Header.jpg" width="50%"></center>

We now need to GET the ID of the NSX Controllers,  to do this we query 'NSX Manager' to get a list of 
'NSX Controllers' it controls.

{% highlight bash %}
GET https://{nsxmgr-ip}/api/2.0/vdn/controller
{% endhighlight %}

This returns data in XML format for ease of reading change to the 'Response Body (Preview)' you will see 
within the XML data the ID of your NSX Controllers take a note of all of these.

<center><img src="/images/RESTclient_Get_Controllers.jpg" width="50%"></center>

We now need to POST syslog configuration to each 'NSX Controller' via the 'NSX Manager' REST API.

{% highlight bash %}
POST https://{nsxmgr-ip}/api/2.0/vdn/controller/{controller-id}/syslog
{% endhighlight %}

With 'Request Body'

{% highlight xml %}
<controllerSyslogServer>
<SyslogServer>{loginsight-ip}</syslogServer>
<port>514</port>
<protocol>UDP</protocol>
<level>INFO</level>
</controllerSyslogServer>
{% endhighlight %}

As REST is a part of HTTP then if all goes well you should get a Success 200.

Repeat POST the same 'Request Body' to 'NSX Manager' changing the {controller-id} value until you have 
posted to all your 'NSX Controllers'.

Once set you can query what 'NSX Controllers' by performing a GET rather than a POST.

{% highlight bash %}
https://{nsxmgr-ip}/api/2.0/vdn/controller/{controller-id}/syslog
{% endhighlight %}

<h3>NSX Edge Configuration</h3>

The NSX Edge's are nice and easy to configure,  albeit via a different console.

Configure NSX Edge Gateway syslog using vCenter Networking &amp; Security,  navigate through to the 
Edge you want to report in.  Change to 'Manage' tab and 'Configuration' menu,  then Edit syslog and 
add the IP address of your Log Insight.

<center><img src="/images/Edge_Syslog.jpg" width="50%"></center>

So that is it,  three different parts of NSX and three different methods for configuring syslog, 
but now all your NSX components should send there syslog's to Log Insight for your viewing pleasure.