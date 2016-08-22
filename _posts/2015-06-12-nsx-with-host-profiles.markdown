---
layout: post
title:  "NSX With Host Profiles"
date:   2015-06-12 16:59:56 +0100
tags:
    - VMware
    - Software Defined Networking
permalink: nsx-with-host-profiles/
---
When trying to use Host Profiles in NSX-V you find that taking configuration from Reference Host captures some NSX specific configuration which is not appropriate to be applied to other hosts.  To get around this once you have captured your host profile from reference host,  select the profile and click to Enable\Disable Profile Configuration.

Work through the menu to clear the check box to disable this configuration in the profile.
<ul>
	<li>Networking configuration &gt; Host Virtual NIC</li>
	<li>Networking configuration &gt; NetStack Instance &gt; VXLAN</li>
	<li>Advanced configuration option, unselect all UserVars.Rmq* variables</li>
</ul>