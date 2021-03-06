# Dell Precision 5510 4K High Sierra Installation

This repository is based on [wmchris' high sierra repo](https://github.com/wmchris/DellXPS15-9550-OSX) for XPS15 9550.

You may refer to [darkhandz's Chinese tutorial](https://github.com/darkhandz/XPS15-9550-High-Sierra/blob/master/README.md) for calculating slider values. 


## Directory Info
* CLOVER: clover configuration files for daily usage
    * This configuration does NOT contain apfs.efi necessary for those who use APFS.
* POST-INSTALL: files needed for post-installation set-ups.


## Installation Pitfalls
* You may have to delete all kexts that start with `Brcm` in CLOVER/kexts if you're stuck in an infinite loop when you attempt to boot into the installer.
* You may have to delete `EmuVariableUefi-64.efi` in CLOVER/drivers64UEFI if you're faced with an error (I can't recall the exact wording of it but it has something to do with being unable to find a file) right after you boot into the installer.



## The Current Status:
### Working:
* Built-in speakers and audio jack
* Samsung SM951 works out of the box, no NVMe kext needed. (Now I'm using Toshiba XG3 with 4K sector)
* Intel HD530 with 4K display
* Typc-C plug
* Bluetooth
* Battery status (charge status, percentage, see Issues #4)
* FakeSMC sensors, also Activity Monitor doesn't crash anymore at Energy tab
* Display brightness
* HWP, with CPU frequency as low as 900MHz.


### Not Working:
* hibernation
* *see also* [Issues](#issues)

Things that are not listed here have not been tested.


## Fix for 4K:
__This has already been achieved by using `CoreDisplayFixup.kext` and `Lilu.kext` (both included in the repo).__ 

1. boot the installer with an invalid ig-platform-id in Clover, e.g. `0x12345678`
2. after the system is installed, disable SIP if necessary
3. apply the pixel clock patch below

```
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay
sudo codesign -f -s - /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay
```

4. reboot with a valid ig-platform-id in Clover, e.g. `0x191b0000`


## Issues <a name="issues"></a>
1. [__Rare__] Random `fs_get_inode_with_hint` errors on reboot.
    * I've had this happened to me several times. I have no idea when and why this happens, but when it does, the fix is to replace apfs.efi in CLOVER with the one in the system. You may find your own `apfs.efi` in `/usr/standalone/i386`. 
    * It's best if you could have a bootable (and of course working) macOS installation on an external disk in case things like these happen. I personally created such one with Carbon Copy Cloner.