# M5510-4K-High-Sierra-Installation

This repository is based on darkhandz's high sierra repo for XPS15 9550.


The BIOS version for my laptop is 1.12.19, thus no need for OsxAptioFix2Drv-64.efi, sliding value calculations and the like.


This repo is far from complete.

### Working:
* Built-in speakers, the earplug seems to be working without any ALCPlugFix
* Samsung SM951 works out of the box, no NVME kext needed.
* Intel HD530 with 4K display
* Thunderbolt hotplug (not very stable)
* Brightness tuning
* HWP

> For the last two, install the two kexts in POST-INSTALL/LE to /Library/Extensions/ and rebuild kernel cache with `sudo kextcache -i /`

### Not Working:
* FakeSMC sensors, but instead Activity Monitor doesn't crash anymore at Energy tab

Things that are not listed here have not been tested.


## To get HD530 work with 4K:
1. boot the installer with an invalid ig-platform-id in Clover, e.g. 0x12345678
2. after the system is installed, disable SIP if necessary
3. apply the perl patch

```
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay


sudo codesign -f -s - /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay
```

4. reboot with a valid ig-platform-id in Clover, e.g. 0x191b0000

## Issues
* Time Machine works with APFS with no problem (I've migrated pretty much everything from previous backups of 10.12.6 with Migration Assitant)
* Couldn't copy and paste for a while after the system is installed. The problem went away by itself after a few reboots (Some have suggested that CoreDisplayFixup breaks copy and paste, but the perl script doesn't)
* iTunes crashes frequently
  * The crashes are mostly (if not all) segmentation faults
  * The crashes do not happen at all with a fake ig-platform-id
