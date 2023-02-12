---
tags: ["nas", "hba", "sas", "lsi", "9300-8i", "truenas", "datahoarder"]
title: "LSI HBA Flashing"
date: "2023-02-10"
author: "Voytek Jarnot"
description: "I recently acquired an LSI 9300-8i Supermicro HBA and needed to flash the firmware for use with TrueNAS... the process was more of a hassle than I'd expected."
---

## LSI HBA IT-mode Firmware Flashing

Oh, what fun we'll have: Flashing an IT-mode firmware to an LSI 9300-8i Supermicro HBA on a UEFI system.

If you want a real guide, read this: https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/

**If my rambling doesn't interest you, [skip to the good stuff](#tldr-lets-flash)**

## Background

Currently the media and other bulk-storage needs of our household are handled by an Intel Rapid Storage RAID 5 array of 4x 3TB disks sitting in my desktop PC.
This arrangement may sound impressive if your concept of bulk storage is limited to finding yet another cheap external hard drive to plug in,
but is laughable to the NAS-enthusiast community. 

Given that it's time for a new PC for myself, I got to thinking about acquiring a 'proper' consumer-grade NAS to untether my day-to-day computer from its
NAS-ish duties. As tends to happen, the rabbit-hole of research led to me learn a bit about Synology and its competitors, work out a game-plan, get an Amazon
shopping cart populated and ready to go, and completely change my mind and reverse course.

Enter the new plan: build myself a new PC, repurpose current PC to serve as a NAS. Enter TrueNAS: https://www.truenas.com/truenas-scale/

## So what?

So the recommended approaches for TrueNAS are
* Buy a box from iXsystems
* Buy a server on eBay, modify as/if needed, and install TrueNAS
 
Those don't meet my goal of repurposing my existing desktop PC, so I'll be going with the **not**-particularly-recommended route of:

* Do what you can to a PC to make it TrueNAS-viable *(for home use, if you're doing this professionally, close this browser tab immediately)*.

So under the guise of doing what's reasonable to make this not suck, I'll be reusing the PC, but hooking up new HDDs to a Host Bus Adapater (HBA) rather
than directly to the SATA ports of my motherboard.

## So why are we flashing firmware?

The canonically-recommended HBAs for TrueNAS are those that utilize LSI chips. A ton of different manufacturers have made HBAs with these chips.
Within that plethora, there's the 92xx series, the 93xx series, and others - this is definitely *not* the place to learn all about LSI HBAs.

I'll be using a Supermicro card with a 9300-8i chip on it, because that's the one I got on eBay. If you're heading down this path, do your research
and confirm that the card you're buying has the chip you want (and you want that chip because you read enough on the TrueNAS forums to know it'll work).

#### But seriously, why?

This is over-simplified to a probably-dangerous point, but here goes:

HBAs operate in one of two (that we're concerned with) modes:
* IR: RAID mode **[no bueno]**
* IT: not RAID mode **[yes bueno]**

For use with TrueNAS, your HBA ***absolutely must*** be in IT mode - and this is set via the firmware that your HBA is running. So - we need to flash 
the IT mode firmware to our HBA.

Also, there this: https://www.truenas.com/community/resources/lsi-9300-xx-firmware-update.145/.
Specifically for the LSI 9300 series, IT mode is not enough, it needs to be IT mode using the specific firmware version available for download at that link.

## Flashing

#### Overview
We need three things: the card (installed), the firmware file, the software to flash the firmware to the card, and a USB stick.
We've covered the first two, regarding the third: you're going to want sas**2**flash for 92xx cards, and sas**3**flash for 93xx cards.

https://www.broadcom.com/support/knowledgebase/1211161501344/flashing-firmware-and-bios-on-lsi-sas-hbas

### TL;DR: Let's Flash!
1. Format your USB stick to FAT32
1. Download EFI Shell: https://github.com/pbatard/UEFI-Shell *(click on Releases and download the ISO)*
    1. Extract the ISO
    1. Make an `efi/boot` directory structure on your USB stick
    1. Copy `bootx64.efi` from the ISO to `/efi/boot/` on the USB stick
1. Download sas3flash.efi (or sas2flash.efi if you're flashing a 92xx card): 
  https://www.broadcom.com/support/knowledgebase/1211161501344/flashing-firmware-and-bios-on-lsi-sas-hbas *(very bottom of the page, note we want the EFI version)*
    1. Copy `sas3flash.efi` to the root directory of your USB stick
1. Acquire firmware - you can probably find this on Broadcom's site, after much googling and reading the TrueNAS forums for the right link for your card. Or use the
version from https://www.truenas.com/community/resources/lsi-9300-xx-firmware-update.145/ if you've got a 9300-xx card.
    1. Copy `SAS9300_8i_IT.bin` *(find the file that matches the chip on your card)* to the root directory of your USB stick
1. Disable secure boot in your BIOS *(on my ASUS mb, this required getting into the Advanced settings)*
1. Boot your USB stick

Now you're in an EFI shell! If it's your first time, yes, I agree, it does look like an OS from the 80s.

Here are the commands I used to flash my firmware:

```
fs0:
sas3flash.efi -listall
sas3flash.efi -c 0 -list
sas3flash.efi -c 0 -f FILENAME
sas3flash.efi -c 0 -list 
```

1. `fs0:` - this is us just changing "directories" to get to our USB stick root
1. `sas3flash.efi -listall` - you're going to want to see your HBA show up here, if it doesn't there's no point in proceeding
1. `sas3flash.efi -c 0 -list` - I assume that, like me, there's only one HBA in your system, so we reference it as `0`
    1. This'll show you what firmware you've currently got, as well as the mode (IR or IT) your HBA is in
    1. While you're here, write down your SAS address, it may come in handy
1. `sas3flash.efi -c 0 -f FILENAME`
    1. This is where the magic happens, in my case FILENAME was `SAS9300_8i_IT.bin`
1. `sas3flash.efi -c 0 -list` - Confirmation: check that your firmware version is what you want it to be, and that you're in IT mode

## Additional Thoughts

These cards are designed for server chassis running in racks in datacenters. I.e., lots and lots of airflow: fans so loud that you'd
never allow them in your house. So they've got heatsinks but no fans on the cards, I've rigged a 40mm fan (https://amzn.to/3If9J1I) *[affiliate]* to mine, 
I recommend you do the same.