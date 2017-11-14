# M5510-4K-High-Sierra-Installation

This repository is based on [darkhandz's high sierra repo](https://github.com/darkhandz/XPS15-9550-High-Sierra) for XPS15 9550.


The BIOS version for my laptop is 1.3.0 (the latest as of Aug 28 2017 on Dell's website), hence OsxAptioFix**2**Drv-64.efi and slider value calculations are necessay to boot into the installer and the OS.


You may refer to [darkhandz's Chinese tutorial](https://github.com/darkhandz/XPS15-9550-High-Sierra/blob/master/README.md) for calculating slider values. 


## Directory Info
* CLOVER: clover configuration files for daily usage
* POST-INSTALL: files needed for post-installation set-ups.


## Installation Pitfalls
* You may have to delete all kexts that start with `Brcm` in CLOVER/kexts if you're stuck in an infinite loop when you attempt to boot into the installer.
* You may have to delete `EmuVariableUefi-64.efi` in CLOVER/drivers64UEFI if you're faced with an error (I can't recall the exact wording of it but it has something to do with being unable to find a file) right after you boot into the installer.



## The Current Status:
### Working:
* Built-in speakers, ALCPlugFix needed for a functional headphone jack.
* Samsung SM951 works out of the box, no NVMe kext needed. (Now I'm using Toshiba XG3 with 4K sector)
* Intel HD530 with 4K display
* Typc-C plug (not very stable)
* Bluetooth (see Issues #3)
* Battery status (charge status, percentage)
* FakeSMC sensors, also Activity Monitor doesn't crash anymore at Energy tab
* Brightness slider && tuning
* HWP, with CPU frequency as low as 900MHz.

> For the last two, install the two kexts in POST-INSTALL/LE to /Library/Extensions/ and rebuild kernel cache with the bash script below.

```
sudo rm -rf /System/Library/Caches/com.apple.kext.caches/Startup/kernelcache  
sudo rm -rf /System/Library/PrelinkedKernels/prelinkedkernel  
sudo touch /System/Library/Extensions && sudo kextcache -u /
```

### Not Working:
* *see also* [Issues](#issues)

Things that are not listed here have not been tested.


## Fix for 4K:
__This has already been achieved by using `CoreDisplayFixup.kext` and `Lilu.kext` (both included in the repo). Plus, the kexts don't break copy && paste, while the patch below does.__ 

1. boot the installer with an invalid ig-platform-id in Clover, e.g. `0x12345678`
2. after the system is installed, disable SIP if necessary
3. apply the pixel clock patch below

```
sudo perl -i.bak -pe 's|\xB8\x01\x00\x00\x00\xF6\xC1\x01\x0F\x85|\x33\xC0\x90\x90\x90\x90\x90\x90\x90\xE9|sg' /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay
sudo codesign -f -s - /System/Library/Frameworks/CoreDisplay.framework/Versions/Current/CoreDisplay
```

4. reboot with a valid ig-platform-id in Clover, e.g. `0x191b0000`

## Fix for dysfunctional earplug after waking up from sleep
__You need to have ALCPlugFix installed first__ -> See [POST-INSTALL/ALCPlugFix](https://github.com/scottsanett/M5510-4K-High-Sierra-Installation/tree/master/POST-INSTALL/ALCPlugFix)

1. install sleepwatcher: `brew install sleepwatcher`
2. start sleepwatcher service: `brew services start sleepwatcher`
3. create a file named: `.wakeup` under you user home directroy: `touch ~/.wakeup`
4. make the file executable: `chmod +x ~/.wakeup`
5. add the following lines to the file

```
#!/bin/bash
pid=`ps -ef | grep "ALCPlugFix"|wc -l`
if [ -gt $pid ] 
then 
    pkill ALCPlugFix
else
    nohup </dev/null /usr/bin/ALCPlugFix &
    sleep 1 s
    pkill ALCPlugFix
fi
```

## Concerning `X86PlatformPluginInjector` and darkwake
For some reasons the OS has been having problems with sleeping. (It's not necessarily a problem with High Sierra. It might have been there since Sierra.) It sporadically wakes itself up (without lighting up the monitor), sometimes it doesn't sleep at all. My guess is that it has something to do with darkwake, and since I don't need it anyway, I might as well disable it altogether. 

The new `X86PlatformPluginInjector.kext` contains a `Mac-A5C67F76ED83108C.plist` under `contents/resources/` that has been taken directly from `/System/Library/Extensions/IOPlatformPluginFamily.kext/Contents/PlugIns/X86PlatformPlugin.kext` with the following modifications:
* Under `FrequenciesVectors`, I changed the second hex from `0d000000` to `09000000` to enable a lowest frequency rate at 900MHz. The value differs for different CPU models and needs to be changed accordingly.
* `Nofication Wake` has been changed from `YES` to `NO`;
* `DarkwakeServices` has been removed.
* `pmset` settings:
	* `hibernatemode` set to 0
	* `ttyskeepawake` set to 0
	* `tcpkeepalive` set to 0


## Concerning `ApplePS2SmartTouchpad.kext`
The author of the kext seems to have postponed the development of this driver until the end of this year. This configuration uses Version 3.6 beta 5, which is the latest as of October 18 2018.

The kext has been modified and tuned for a better usability, as some of the gestures are frequently misinterpreted.

The kext now supports these gestures:
### Click
* bottom click: middle click
* 1 finger click: left lick
* 2 finger click: right click
* 3 finger click: preview (files, webpages, look up words in dictionary)
* 4 finger click: hide current window
* 5 finger click: close current window (or current tab in multi-tab applications, might crash Safari as well)

### Pinch
* 5 finger pinch: quit application

### Press
* 4 finger press: show application windows
* 5 finger press: show launchpad (keyboard shortcut for Show Launchpad needs to be mapped to F12)
(3 finger press weirdly doesn't work in this version on my laptop)

### Swipe
* 3 finger swipe left: web browser goes forward
* 3 finger swipe right: web browser goes backward
* 4 finger swipe left: go to next desktop
* 4 finger swipe right: go to previous desktop
* 4 finger swipe up: maximum window
* 4 finger swipe down: minimize window

### Edge Swipe
* right edge swipe left: notification center (keyboard shortcut for `Show Notification Center` needs to be mapped to CTRL CMD N)
* left edge swipe right: application switcher
* top edge swipe down: mission control
* bottom edge swipe up: show desktop

### Rotation
Due to the limitation of the kext itself, the gesture doesn't look like a rotation. For it to work, one needs to set a finger (I'd suggest using index as it's easier) on the touchpad, and flick the middle or the ring finger, up for left rotation, down for right rotation.
Rotations are mapped to `CMD + L` and `CMD + R`, hence right rotation refreshes webpages in browsers.


## Issues <a name="issues"></a>
1. __[Seemingly Fixed]__ iTunes crashes frequently. 
    * The crashes do not happen at all with a fake ig-platform-id. 
    * I'm temporarily using Swinsian as an replacement (for music management, not iOS device manager).
2. Random `fs_get_inode_with_hint` errors on reboot.
    * I've had this happened to me several times. I have no idea when and why this happens, but when it does, the fix is to replace apfs.efi in CLOVER with the one in the system. You may find your own `apfs.efi` in `/usr/standalone/i386`. 
    * It's best if you could have a bootable (and of course working) macOS installation on an external disk in case things like these happen. I personally created such one with Carbon Copy Cloner.
3. Bluetooth is semi-functional.
    * It is possible to transfer files between devices (I specified file transfer as I've tested only that), but once one tries to turn it off, it not only stays on, but file transter stops working as well, which only a reboot can fix.


