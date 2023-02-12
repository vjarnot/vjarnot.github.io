---
tags: ["nas", "hba", "lsi", "9300-8i"]
title: "Supermicro AOC-S3008L-L8E + Noctua NF-A4x20"
date: "2023-02-12"
author: "Voytek Jarnot"
description: "LSI-based HBAs are designed for the airflow of a servier chassis, not a repurposed desktop PC. They run hot, and - in my opinion - should get a little cooling assistance."
---

## Supermicro LSI HBA Fan Addition

When I initially installed my eBay-acquired HBA to flash its firmware, I noticed after a bit that the heatsink was *very* hot to the touch. 
My infrared thermometer read the heatsink at over 135° F, so the heatsink was approaching 60° C; the chip will be - obviously - hotter than the heatsink.
This was at idle not no disks plugged in. I've read that these LSI chips would ultimately prefer a life at or below 55° C, so I've added a fan to my HBA.

I've seen 3D-printed mounts, I've seen screws - both into the heatsink and through the heatsink, and I've seen zipties. 
I don't think I've seen someone do their zipties as I have - around the heatsink, not the HBA, so I thought I'd throw up some pictures in case it helps anyone.

![Supermicro AOC-S3008L-L8E + Noctua NF-A4x20](images/supermicro-hba-noctua-1.jpg)

![Supermicro AOC-S3008L-L8E + Noctua NF-A4x20](images/supermicro-hba-noctua-2.jpg)

![Supermicro AOC-S3008L-L8E + Noctua NF-A4x20](images/supermicro-hba-noctua-3.jpg)
