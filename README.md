# M5510-4K-High-Sierra-Installation

This repository is based on darkhandz's high sierra repo for XPS15 9550.


The BIOS version for my laptop is 1.12.19, thus no need for OsxAptioFix2Drv-64.efi, sliding value calculations and the like.


This repo is far from complete. At the current stage:

Working:
* Sound card (earplug not tested)
* Samsung SM951 works out of the box, no NVME kext needed.
* Intel HD530 with 4K display
* Brightness

To get HD530 work with 4K:
1. boot the installer with an invalid ig-platform-id in Clover, e.g. 0x12345678
2. disable SIP if necessary
3. apply the perl patch

```
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay


sudo codesign -f -s - /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay

```

4. reboot with a valid ig-platform-id in Clover, e.g. 0x191b0000

Things that are not listed here have not been tested.
