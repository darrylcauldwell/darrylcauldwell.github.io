+++
title = "VMware Cloud Foundation Lifecycle Management - VCF LCM"
date = "2021-04-27"
description = "VMware Cloud Foundation Lifecycle Upgrades - VCF LCM"
tags = [
    "vcf",
    "lcm"
]
categories = [
    "vsphere"
]
thumbnail = "clarity-icons/code-144.svg"
+++
VMware Cloud Foundation provides automated initial deployment and ongoing lifecycle management upgrades. Here I am exploring lifecycle management upgrades and exploring ways to make this as efficient as possible.

## Bundle Download

The SDDC Manager appliance controls the Cloud Foundation deployment. To upgrade the environment(s) requires the software packages. With each Cloud Foundation release, the required software packages are formed into bundles. The bundles are made available to download to licensed users via their 'My VMware' account. 

There are two methods of downloading the bundles. SDDC Manager can be configured with 'My VMware' account and the LCM service then polls the VMware depot to access update bundles. Alternatively, the Bundle Transfer utility can be used to manually download the bundles from the depot and then upload them to SDDC Manager.

There is little control and limited UI visibility of bundle downloads using SDDC Manager. This is fine if you advanced plan and configure the account in time to poll available bundles and initiate downloads. The LCM service writes two files that can be useful in understanding progress.

```bash
/var/log/vmware/vcf/lcm/lcm.log
/var/log/vmware/vcf/lcm/lcm-debug.log
```

When you have multiple Cloud Foundation deployments it can be more efficient to use the Bundle Transfer utility to download once and distribute internally. The Bundle Transfer utility is also useful to gain a little more control over the download process.

The LCM Bundle Transfer utility is shipped with the SDDC Manager appliance. The utility can be used to download various bundle versions. The utility has two elements, to ensure the utility downloads the appropriate bundles for your deployment the first extract environmental metadata from SDDC Manager.

```bash
## SSH to SDDC Manager
## Outputs required metadata in files
## ~/markerFile
## ~/markerFile.md5

cd /opt/vmware/vcf/lcm/lcm-tools/bin
./lcm-bundle-transfer-util --generateMarker
```

The second step takes the metadata as input and performs the download. This step requires Java 8 (or later) to be installed but can be run on either Windows or Linux.

```bash
mkdir F:\3.5.1\bundleDownload
scp -rp vcf@nest-1-sddc-manager.cork.local:/opt/vmware/vcf/lcm/lcm-tools F:\3.5.1\bundleDownload
scp -p vcf@nest-1-sddc-manager.cork.local:/home/vcf/markerFile F:\3.5.1\bundleDownload
scp -p vcf@nest-1-sddc-manager.cork.local:/home/vcf/markerFile.md5 F:\3.5.1\bundleDownload
cd F:\3.5.1\bundleDownload\lcm-tools\bin
lcm-bundle-transfer-util --download --outputDirectory F:\3.5.1\bundleDownload -depotUser {your my.vmware.com username} --markerFile F:\3.5.1\bundleDownload\markerFile --markerMd5File F:\3.5.1\bundleDownload\markerFile.md5
```

Once the bundle downloads are complete we can copy them to SDDC Manager and import them into the repository.

```bash
## From Windows device used to download
scp -pr F:\3.5.1\bundleDownload vcf@Snest-1-sddc-manager.cork.local:/nfs/vmware/vcf/nfs-mount/
## SSH to SDDC Manager and set permissions
chmod -R 0777 /nfs/vmware/vcf/nfs-mount/bundleDownload
cd /opt/vmware/vcf/lcm/lcm-tools/bin
./lcm-bundle-transfer-util --upload --bundleDirectory /nfs/vmware/vcf/nfs-mount/bundleDownload
```
## Update Pre-check

Once the bundles are uploaded successfully to the repository we can look towards installing them. Upgrades can fail to apply if the environment isn't healthy. Upgrades can take a while to apply so it is better to avoid failure and rollback. A series of environmental health pre-checks are included which can be run to avoid failure to successfully deploy bundles.

* Domain Manager
    * Common services availability check
    * Inventory status check
    * SDDC Manager VM database data directory has enough disk space
    * SDDC Manager VM has enough disk space
    * SDDC Manager VM log directory has enough disk space
    * SDDC Manager VM system directory has enough disk space
    * Service availability check
* SDDC Manager UI
    * Inventory status check
    * SDDC Manager VM database data directory has enough disk space
    * SDDC Manager VM has enough disk space
    * SDDC Manager VM log directory has enough disk space
    * SDDC Manager VM system directory has enough disk space
    * Service availability check
* Operations Manager
    * Common services availability check
    * Inventory status check
    * SDDC Manager VM database data directory has enough disk space
    * SDDC Manager VM has enough disk space
    * SDDC Manager VM log directory has enough disk space
    * SDDC Manager VM system directory has enough disk space
    * Service availability check
* Platform Services Controllers
    * PSC health
    * Credential availability
    * Inventory status check
    * LCM bundle repo available
    * NTP sync
    * PSC inventory status check
    * PSC SSO connection check
    * PSC vmdir service check
    * LCM bundle repo free space
    * SDDC Manager VM has enough disk space
* NSX Manager
    * Backup availability
    * Credential availability
    * Enter maintenance dryrun check
    * Inventory status check
    * LCM bundle repo available
    * NSX audit check
    * NSX manager inventory status check
    * NTP sync
    * NSX for vSphere plugin connectivity check
    * NSX precheck init 
    * LCM bundle repo free space
    * SDDC Manager VM has enough disk space   
* vCenter
    * vCenter health
    * Credential availability
    * Inventory status check
    * LCM bundle repo available
    * NTP sync
    * LCM bundle repo free space
    * SDDC Manager VM has enough disk space
    * vCenter inventory status
    * VIM API
* Common Services
    * Credential availability
    * Inventory status check
    * LCM bundle repo available
    * NGINX check
    * NTP sync
    * LCM bundle repo free space
    * SDDC Manager VM has enough disk space
    * SDDC Manager VM log directory has enough disk space
    * SDDC Manager VM system directory has enough disk space
    * Service availability check
* vSAN
    * Advanced configuration sync health
    * CLOMD liveness health
    * Cluster health
    * Encryption health
    * HCL age
    * HCL health
    * Network health
    * Object health
    * Health summary
* LCM
    * Depot connection
    * Depot user
    * LCM bundle repo available
    * LCM database schema version
    * LCM directory permissions
    * SDDC Manager VM database data directory has enough disk space
    * SDDC Manager VM has enough disk space
    * SDDC Manager VM log directory has enough disk space
    * SDDC Manager VM system directory has enough disk space
* ESXi Hosts
    * Config check
    * Enter maintenance dryrun check
    * Inventory status
    * NSX sync
    * VIM API
    * LCM bundle repo available
    * LCM bundle repo freespace
    * Local filesystem check
    * vCenter Connection
    * VUM Health

Each check returns health state green, yellow or red if an issue is detected some information on impact and remediation is made available. Sometimes the information returned isn't perfect. The LCM pre-checks output to two files which can be useful in the understanding issue.

```bash
/var/log/vmware/vcf/lcm/lcm.log
/var/log/vmware/vcf/lcm/lcm-debug.log
```

Not all issues require full remediation. In my nested lab environment, the vSAN HCL returns a red issue which can never be resolved. I can either just ignore the error and proceed or we can look to disable the particular check in the configuration.

```bash
vi /opt/vmware/vcf/lcm/lcm-app/conf/application-prod.properties
```

## Bundle Apply

When you are happy with environmental health next to move to apply bundles. To move between Cloud Foundation versions can require applying multiple bundles.  For example, when moving from 3.5.1 to 3.7.1 following bundles are required.

* SDDC Manager
    * 3.5.1-12050813 to 3.7.0-12696026 (bundle-9775.tar)
    * 3.7.0-12696026 to 3.7.0-12698020 (bundle-9776.tar)
    * 3.7.0-12698020 to 3.7.1 (bundle-11811)
    * 3.7.0-12698020 to 3.7.1 part 2 (bundle-11813)
* Management domain
    * NSX 6.4.1 to 6.4.4 (bundle-6712)
    * vCenter 6.5 to 6.7.0-10244745 (bundle-6713)
    * vCenter 6.7.0-10244745 to vCenter 6.7.0-11726888 (bundle-9777)
* Workload domain
    * NSX 6.4.1 to 6.4.4 (bundle-6712)
    * vCenter 6.5 to 6.7.0-10244745 (bundle-6713)
    * vCenter 6.7.0-10244745 to vCenter 6.7.0-11726888 (bundle-9777)

Some of the bundles can take a long time to apply and the UI isn't very verbose. It is very useful to hold an SSH session to SDDC Manager to tail the summary log file to view progress when applying bundles.

```bash
tail -f /var/log/vmware/vcf/lcm/lcm.log
```

The main workflow process spawns other processes so those main upgrade log files sometimes aren’t that verbose and often don’t help identify the issue. It is useful to understand that the spawned processes also log to folder UUID of the bundle apply task. The different bundles create sub-folders and write various logs examples.

```bash
/var/log/vmware/vcf/lcm/upgrades/<upgrade_ID>/thirdparty/sddc-migration-app/logs
/var/log/vmware/vcf/lcm/upgrades/<upgrade_ID>/sddcmanager-migration-app/logs/sddcmanager_migration_app_upgrade.log
```

## Skip Level Upgrades

As we found before moving between Cloud Foundation versions can lead to applying multiple bundles to the same product. The skip-level upgrade tool was introduced to allow easier uplift to 3.10.1.2 or 3.10.2. For NSX for vSphere based workload domains can uplift to 3.10 from 3.5 and for NSX-T can uplift from 3.7.1.

Here I am using the skip level upgrade tool to uplift a 3.5.1 NSX-V environment to 3.10.2 some twenty six bundles.

![Skip Level Download](/images/vcf-lcm-skip-download.png)

## Apache Cassandra

The application uses a Apache Cassandra database to store data. When troubleshooting it can be useful to look through the contents.

```bash
## SSH to SDDC manager and open client
cqlsh --cqlversion=3.4.4 127.0.0.1 9042

## Get all keyspaces
describe keyspaces;

## Change to lcmkeyspace
use lcmkeyspace;

## Get all lcmkeyspace tables
describe tables;

## Get all host entries from upgrade_activity_log table in json format
select json * from upgrade_activity_log;
```
