+++
title = "Setup Hanlon"
date = "2014-04-04"
description = "Configuring Project Hanlon On CentOS 6.5"
tags = [
    "hanlon"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
+++
Since the company I work for CSC got a new CEO then CTO its changed direction and the latest change has been to [participate with the Open Source community](http://www.vdatacloud.com/blogs/2014/05/22/finding-value-in-opensource/).  With this news shortly followed an updated version of a tool our new CTO [Dan Hushon](https://twitter.com/DanHushon) worked on at EMC with [Nicholas Weaver](https://twitter.com/lynxbat) it was formerly called [Razor](https://github.com/puppetlabs/Razor") and the new fork is [Hanlon](https://github.com/csc/Hanlon).

I chose CentOS 6.5 64bit to host Hanlon and configured this as a VM with two vNICs one normal internet facing and one on a new private vSwitch with no uplinks,  the private is where DHCP\PXE\TFTP will be setup on and where the ESXi VMs will hopefully get built.  All these sit on an nested ESXi 5.0 configured with 12GB of vRAM and access to 1TB of NFS disk running inside VMware Fusion 6 on a Mac.

I don't work with Linux so some of my commands might be sub optimal, I'll try and record all I use here for reference, and if you see anything I'm doing badly please do say.

First task to the new VM is to start VMware tools installer which mounts the correct ISO,  then install the tools via putty session (or via console).

```bash
yum -y install perl
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cp /mnt/cdrom/VMwareTools-*.tar.gz /tmp
tar -zxf /tmp/VMwareTools-*.tar.gz -C /tmp
cd /tmp/vmware-tools-distrib/
./vmware-install.pl --default
rm -f /tmp/VMwareTools-*.tar.gz
rm -rf /tmp/vmware-tools-distrib

# I then changed from DHCP to static IP addressing,  to note shown here are only the lines I changed or added the rest I left as they populated by OS

vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=none
NETWORK=&lt;Internet Facing Network Address&gt;
NETMASK=&lt;Internet Facing Subnet Mask&gt;
IPADDR=&lt;Internet Facing IP Address&gt;
vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=none
NETWORK=&lt;Private Network Address&gt;
NETMASK=&lt;Private Subnet Mask&gt;
IPADDR=&lt;Private IP Address&gt;

# My next port of call was to add a DHCP server to the VM, I chose ISC as that was recommended for Razor and appeared simple and well [documented](http://prefetch.net/articles/iscdhcpd.html).

## Install DHCP
yum install dhcp

## Change DHCP that to listen on eth1
vi /etc/sysconfig/dhcpd

DHCPDARGS=eth1

## Create a new config file
vi /etc/dhcp/dhcpd.conf

## Populate with following (altering with your ID address ranges)
subnet 172.25.1.0 netmask 255.255.255.0 {
range 172.25.1.20 172.25.1.50;
option tftp-server-name "172.25.1.10";
if exists user-class and option user-class = "iPXE" {
filename "hanlon.ipxe";
} else {
filename "undionly.kpxe";
}
next-server 172.25.1.10;
}

## Start DHCP using
/etc/init.d/dhcpd start
```

Test ISC DHCP and option by creating a VM with no OS,  and with only a NIC on the 
private network and it should now pickup a PXE address.

![Hanlon PXE Boot](/images/hanlon-PXE-Boot.gif)

It will fail as there is no PXE or TFTP server setup yet on the Hanlon server. So next step is to add a PXE and TFTP server,  so first task is to install it.

```bash
yum -y install tftp-server tftp

## Enable TFTP
sed -i 's/disable\t\t\t= yes/disable\t\t\t= no/g' /etc/xinetd.d/tftp
service xinetd restart

## Install Java
yum -y install java-1.7.0-openjdk

## Create MongoDB Repo config file
vi /etc/yum.repos.d/mongodb.repo

## Then populate new file with
[mongodb]
name=MongoDB Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
gpgcheck=0
enabled=1

## Install MongoDB from repo
yum -y install mongodb-org

## Install Torquebox Ruby Application Platform
curl -L -O http://torquebox.org/release/org/torquebox/torquebox-dist/3.0.1/torquebox-dist-3.0.1-bin.zip
yum -y install unzip
unzip torquebox-dist-3.0.1-bin.zip -d $HOME
echo 'export TORQUEBOX_HOME=$HOME/torquebox-3.0.1' &gt;&gt; ~/.bashrc
echo 'export JBOSS_HOME=$TORQUEBOX_HOME/jboss' &gt;&gt; ~/.bashrc
echo 'export JRUBY_HOME=$TORQUEBOX_HOME/jruby' &gt;&gt; ~/.bashrc
echo 'export PATH=$JRUBY_HOME/bin:$PATH' &gt;&gt; ~/.bashrc
exec $SHELL -l

## Download Hanlon
mkdir /opt/hanlon
cd /opt/hanlon
yum -y install git
git clone https://github.com/csc/Hanlon.git

## Install Requisite Ruby Gems for Hanlon
cd /opt/hanlon/Hanlon
bundle install
gem update --system

## Just need now to finalise the configuration of the Hanlon Microkernel, configure TFTP and  iPXE.

## Install GCC
yum -y install gcc gcc-c++

## Download iPXE & Make Files
git clone https://github.com/ipxe/ipxe.git
curl -O https://gist.githubusercontent.com/jcpowermac/7cc13ce51816ce5222f4/raw/4384911a921a732e0b85d28ff3485fe18c092ffd/image_comboot.patch
yum -y install patch genisoimage
patch -p0 < image_comboot.patch
cd ipxe/src
make

## Move new iPXE files to correct place for TFTP
cd bin
cp undionly.kpxe /var/lib/tftpboot/undionly.kpxe
cp ipxe.pxe /var/lib/tftpboot/ipxe.kpxe
cp ipxe.iso /var/lib/tftpboot/ipxe.iso

## Get Syslinux files and move to correct place
curl -O https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.02.tar.gz
tar -zxvf syslinux-6.02.tar.gz --strip-components 3 -C /var/lib/tftpboot syslinux-6.02/bios/core/pxelinux.0
tar -zxvf syslinux-6.02.tar.gz --strip-components 4 -C /var/lib/tftpboot syslinux-6.02/bios/com32/menu/menu.c32

## Create Hanlon.pxe
yum -y install wget
wget https://github.com/csc/Hanlon-Microkernel/releases/download/v1.0/hnl_mk_prod-image.1.0.iso<
/opt/hanlon/Hanlon/cli/hanlon image add -t mk -p hnl_mk_prod-image.1.0.iso
/opt/hanlon/Hanlon/cli/hanlon config ipxe > /var/lib/tftpboot/hanlon.ipxe

## Configure TFTBoot
mkdir /var/lib/tftpboot/pxelinux.cfg
cat > /var/lib/tftpboot/pxelinux.cfg/default <<EOF
default menu.c32
prompt 0
menu title Hanlon Boot Menu
timeout 50
f1 help.txt
f2 version.txt
label hanlon-boot
menu label Automatic hanlon Node Boot
kernel ipxe.lkrn
append initrd=hanlon.ipxe
label boot-else
menu label Bypass hanlon
localboot 1
EOF>>

## Bind TFTP to eth1
vi /etc/xinetd.d/tftp
add bind = <ipaddress> of eth1
chkconfig dhcpd on
service xinetd restart

## Test TFTP is working
tftp localhost4
get hanlon.ipxe
quit

## If TFTP Times Out Try Disable Firewall
service iptables stop
```

At this point you should be able to create a test VM bound to same Host Only network and it should pick up a DHCP address then connect to TFTP server and download and boot an image.