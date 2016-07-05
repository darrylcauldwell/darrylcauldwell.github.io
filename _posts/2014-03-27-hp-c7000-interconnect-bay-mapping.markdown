---
layout: post
title:  "HP C7000 Interconnect Bay Mapping"
date:   2014-03-27 16:59:56 +0100
tags:
    - Hardware
permalink: hp-c7000-interconnect-bay-mapping
---
Here I address the most common query people have when using HP C7000 chassis, namely how do the bay 
mappings work.

<H3>Half Height Blades</H3>
HP supply blades which occupy single or double slots,  the single height blades map in the following way.

OnBoard NIC1        Map to Interconnect Bay 1  
OnBoard NIC2        Map to Interconnect Bay 2  
Mezz1 HBA1        Map to Interconnect Bay 3  
Mezz1 HBA2        Map to Interconnect Bay 4  
Mezz2 NIC1        Map to Interconnect Bay 5  
Mezz2 NIC2        Map to Interconnect Bay 6  
Mezz2 NIC3        Map to Interconnect Bay 7  
Mezz2 NIC4        Map to Interconnect Bay 8  

<center><img src="/images/image4.png" width="50%"></center>

<H3>Full Height Blades</H3>
For dual height blade server such as BL680c the mappings work the same way although each server 
has two mappings to each for example slot one maps to ports one and nine in each interconnect bay 
as opposed to just bay port one if in slot one and port nine if in slot nine.

<center><img src="/images/c7000_full.gif" width="50%"></center>
