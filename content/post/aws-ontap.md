+++
title = "NetApp Cloud OnTap For AWS"
date = "2014-12-12"
description = "NetApp Cloud OnTap For AWS"
tags = [
    "aws",
    "netapp"
]
categories = [
    "storage"
]
thumbnail = "clarity-icons/code-144.svg"
+++
NetApp has often been a pioneer of bleeding edge concepts in storage. When designing data ONTAP and the WAFL file system they were the first to look at creating a virtual control plane for data. They also pioneered fully non disruptive upgrades when the introduced clustered data ONTAP.  They then introduced ONTAP Edge for providing the really cool features of data ONTAP for non NetApp SANs. Recently the pioneering spirit has been continuing with the launch of Cloud ONTAP for AWS which takes the clustered data ONTAP features to overlay Amazon EC2 storage.

There are hundreds of business use cases which spring to mind when you begin to consider what this offers, however the total killer feature for this is using snap mirror to extend your data between private cloud and public cloud.

![NetApp Cloud OnTap for AWS](/images/aws-ontap.jpg)

Using the public cloud makes great sense for standing up solutions quickly and without the up front capital expenditure and instead the operational usage charging. As your new product grows and becomes more popular it maybe your operational charges out weight the cost savings and you then want to bring it to private cloud.

Most applications have great agility for the application, simply stand up a new OS in other cloud provider and install application.  However application data agility is, until now, much harder to manage.

In this example you would simply build your private cloud and new application tier and use snap mirror to bring your data to your datacenter and keep it in sync with the application still running.  When all sync'd you would cutover by breaking the mirror all could be seam less to your customers.

I expect as well that while currently this is only for EC2 storage, if it was to also interface into S3 and glacier you could also move your data tiering

While I love NetApp I'm not a storage admin,  and while the ONTAP simulator is useful at $1.50 per hour if I ever wanted to exercise my skills I could just spin up a clustered ONTAP AMI and away I would go then release it to save the recurring charge.

While there is more detailed info on the [NetApp website](http://www.netapp.com/us/system/pdf-reader.aspx?cc=us&amp;m=ds-3618.pdf&amp;pdfUri=tcm:10-127470), a superb way to see this in action is to sign up for free and look at [test drive](https://poc.netapp.com/cloud/testdrive/)where you get to spin up a AMI on NetApp's AWS account and look around.