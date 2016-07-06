---
layout: post
title:  "NSX Backup and Restore"
date:   2016-05-20 16:59:56 +0100
tags:
    - Software Defined Networking
permalink: nsx-backup-restore
---
As part of study for VCIX-NV I've given myself task of exploring in my new home lab all parts of 
NSX which I don't use at work. One of these things is NSX Backup and Restore,  its not that we 
don't use these function but that its relatively high impacting if it goes wrong.  To investigate 
this I came up with a list of networking configuration to test backup and restore of and the steps 
to I followed to perform each backup and restore type.

<H3>Scenario 1 - Distributed Firewall</H3>
To backup and restore distributed firewall rules is relatively straight forwards to do.  I'm starting 
with no rules created.

<center><img src="/images/BlankRuleset.jpg" width="50%"></center>

Then create a new section.

<center><img src="/images/BandR.jpg" width="50%"></center>

Add some rules for example any to WEB on HTTP Service and WEB to DB on SQL Service.

<center><img src="/images/testRules.jpg" width="50%"></center>

Click Export and save these as a file.

Delete the WEB to DB rule.

To perform a restore we do this from the Saved Configuration tab,  we see that there are only auto 
saved backups.

<center><img src="/images/Restore1.jpg" width="50%"></center>

In order to restore from the file backup we need to import this into the Saved Configurations.

<center><img src="/images/Restore2.jpg" width="50%"></center>

Now we can change back to the Configuration tab and use 'Load Saved Configuration' option to restore 
the exported file based backup.

<center><img src="/images/Restore3.jpg" width="50%"></center>

The deleted rule then gets added back.

<H3>Scenario 2 - NSX Manager</H3>
To backup and restore NSX Manager it depends on having access to a FTP or SFTP to store the configuration in.

Previously I'd created a CentOS7 virtual machine template,  I deployed one of these and then followed 
this procedure to configure FTP.

Configure the FTP server settings in NSX.

<center><img src="/images/nsxBackup.jpg" width="50%"></center>

Once backup location is configured click Backup to create a backup.

Check the FTP server and it should have a folder called nsx created and within a file prefixed with 
nsx followed by the date and time stamp.

Make a change to NSX configuration,  for example assign the NSX Manager the primary role.

<center><img src="/images/PrimaryRole.jpg" width="50%"></center>

Check role is applied.

<center><img src="/images/Primary.jpg" width="50%"></center>

If we now restore the configuration from the NSX Backup we should see the NSX Manager revert to 
running the Standalone role.

Select the backup created prior to making the configuration change and click Restore.  The NSX Manager 
will restore and then restart the Management Services so will take a minute or two. We can then check the role is reverted to Standalone.

<center><img src="/images/Standalone.jpg" width="50%"></center>

<H3>Scenario 3 - vSphere Distributed Switch</H3>
I have in lab a vSphere Distributed Switch configured with some Distributed Portgroups and Logical 
Switches created on.  Otherwise you might need to create some configuration.

Use the context menu of the vDistributed Switch to Export the configuration to a file.

<center><img src="/images/Export-vDS.jpg" width="50%"></center>

This creates a compressed zip file,  most files within this are not readable except for an xml file 
which appears to be a manifest of the zip file.

To test that the configuration is restored add an additional distributed portgroup named TestRestore1.

<center><img src="/images/TestRestore.jpg" width="50%"></center>

Use the context menu of the vDistributed Switch to Restore the configuration from the file.  Interesting 
we find that the Portgroup created post backup is not removed.

If we export the configuration again to include the new TestRestore1 Portgroup.  Then delete the 
TestRestore1 Portgroup.  Then Restore the configuration we find the TestRestore1 Portgroup is added back.