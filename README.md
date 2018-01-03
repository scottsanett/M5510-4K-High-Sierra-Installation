# Dell Precision 5510 4K High Sierra Installation

This repository is based on [wmchris' high sierra repo](https://github.com/wmchris/DellXPS15-9550-OSX) for XPS15 9550.


The BIOS version for my laptop is 1.5.0 (the latest as of Dec 20 2017 on Dell's website), hence OsxAptioFix**2**Drv-64.efi and slider value calculations are necessay to boot into the installer and the OS.


You may refer to [darkhandz's Chinese tutorial](https://github.com/darkhandz/XPS15-9550-High-Sierra/blob/master/README.md) for calculating slider values. 


## Directory Info
* CLOVER: clover configuration files for daily usage
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
### Update
I have replaced this kext with `VoodooPS2Controller` due to stability issues. This kext is more feature-rich and has better support for scrolling, but its debugged nature floods the Console and it seems buggy in the following aspects:

1. the keyboard is stuck (having to wait a few seconds to be able to input anything) during caps switch after a few days of not rebooting the machine; this can be (temporarily) fixed by sleeping.
2. if you attempt to clean the touchpad and mop all over it with the machine still awake, the touchpad stops working altogether; only a reboot is able to fix this.

`VoodooPS2Controller` seems a little bit stabler and more responsive, but the scrolling and gesture support are far from satisfactory. Thus I'm using the touchscreen with a paid driver that supports native macOS gestures as a replacement.

If you decided to use this over `VoodooPS2Controller`, replace `VoodooPS2Controller.kext` in the kext directory with `ApplePS2SmartTouchpad.kext`, which can be found in the POST-INSTALL.

### Info
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


### An Alternative Solution to Touchpad Kexts
I have known about the touchscreen drivers for macOS by touch-base for quite a while and yesterday I decided to give it a try. The driver enables multi-touch support for the touchscreen (macOS supports only single touch natively and it works like a mouse so no surprise there), and it allows personalizations for many gestures that call macOS APIs (pinch zooms, smart zooms, smooth scrolling, rotation, switching between fullscreen apps/show launchpad/show desktop/etc. animation that follows the fingers rather than a workaround that invokes the corresponding keyboard shortcut, etc.), which no touchpad kext that I know of is capable of doing. It is by no means cheap ($105 for a home license), but it works well. This could be a solution for those unsatisfied with all the touchpad kexts available as it provides a native macOS trackpad feel. [demo(YouTube)](https://www.youtube.com/watch?v=rOvU1TdL2-8)


## Issues <a name="issues"></a>
1. __[Fixed]__ iTunes crashes frequently. 
    * The crashes do not happen at all with a fake ig-platform-id. 
    * I'm temporarily using Swinsian as an replacement (for music management, not iOS device manager).
2. __[Rare]__ Random `fs_get_inode_with_hint` errors on reboot.
    * I've had this happened to me several times. I have no idea when and why this happens, but when it does, the fix is to replace apfs.efi in CLOVER with the one in the system. You may find your own `apfs.efi` in `/usr/standalone/i386`. 
    * It's best if you could have a bootable (and of course working) macOS installation on an external disk in case things like these happen. I personally created such one with Carbon Copy Cloner.
3. __[Fixed]__ Bluetooth is semi-functional.
    * It is possible to transfer files between devices (I specified file transfer as I've tested only that), but once one tries to turn it off, it not only stays on, but file transter stops working as well, which only a reboot can fix.
4. __[Rare]__ The battery menubar icon does not appear on startup/reboot
    * Changing `PublicBatteryFactor` from `YES` to `NO` in `X86PlatformPluginInjector.kext` fixed the issue of battery percentage not showing correctly (it doesn't sync with the data in HWMonitor), with the price that the built-in battery icon in menubar does NOT upgrade itself. I'm using iStat Menus as a workaround, which does upgrade the values (and displays the remaining time if you like) correctly.
    * Half-dimming the monitor when unplugged does not work properly on startup/reboot: the screen barely dims. A workaround is using execute a `pmset` command that doesn't really need to change any existing setting. The command I use is `sudo pmset -a ttyskeepawake 0`. I set it to execute on startups with a delay of 30 seconds with a modified [launchd-oneshot](https://github.com/cybertk/launchd-oneshot) , a shell script that makes it possible to execute scripts with superuser privilege on startup. It does automatic cleanup after the task is completed, which is not what I want, so I removed the code concerning cleanup in the shell script. 


