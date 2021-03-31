+++
title = "HP C7000 Interconnect Bay Mapping"
date = "2014-03-27"
description = "HP C7000 Interconnect Bay Mapping"
tags = [
    "hpe"
]
categories = [
    "automation"
]
thumbnail = "clarity-icons/code-144.svg"
featured = false
+++
Here I address the most common query people have when using HP C7000 chassis, namely how do the bay 
mappings work.

## Half Height Blades
HP supply blades which occupy single or double slots,  the single height blades map in the following way.

| Device | Interconnect Bay Mapping |
| --- | --- |
| OnBoard NIC1 | Bay 1 |
| OnBoard NIC2 | Bay 2 |
| Mezz1 HBA1 | Bay 3 |
| Mezz1 HBA2 | Bay 4 |
| Mezz2 NIC1 | Bay 5 |
| Mezz2 NIC2 | Bay 6 |
| Mezz2 NIC3 | Bay 7 |
| Mezz2 NIC4 | Bay 8 |

![Half Height Blade Mappings](/images/hpe-c7000-mapping-half.png)

## Full Height Blades
For dual height blade server such as BL680c the mappings work the same way although each server has two mappings to each for example slot one maps to ports one and nine in each interconnect bay as opposed to just bay port one if in slot one and port nine if in slot nine.

![Full Height Blade Mappings](/images/hpe-c7000-mapping-full.gif)
