How to create image and restore it under macos high sierra 10.13.5
===

diskutil unmountDisk /dev/disk1  
hdiutil create -srcdevice /dev/disk0s2 -format UDZO /Volumes/Test\ HD/container.dmg  

is same as:  
hdiutil create -srcdevice /dev/disk1 -format UDZO /Volumes/Test\ HD/container.dmg  
hdiutil create -layout GPTSPUD -partitionType Apple_APFS -srcdevice /dev/disk0s2 -format UDZO /Volumes/Test\ HD/container.dmg  
hdiutil create -layout GPTSPUD -partitionType Apple_APFS -srcdevice /dev/disk1 -format UDZO /Volumes/Test\ HD/container.dmg  

// asr imagescan --source /Volumes/Test\ HD/container.dmg  
// => NG (internal error)  

// asr restore --source /Volumes/Test\ HD/container.dmg --target /dev/disk2 --erase  
// => NG (has no container disk)  

hdiutil attach /Volumes/Test\ HD/container.dmg  
=> attached as diskXX  
asr restore --source /dev/diskXX --target /dev/disk2 --erase  
=> OK (however, apfs_invert command required, see below.)  

### explain...

"container image as device" contains meeningfull whole APFS container (including Preboot, Recovery, VM).  
However, this type of image has no partition map information and volume title information.  
So, "asr imagescan" and "asr restore" goes to fail.  
We have to mount image first by "hdiutil attach", then use "asr restore" command.  

When we do this process in booted by macos USB installer, last process of "asr restore" will be fail.  
Because last process is inverting APFS container process, but, macos USB installer has no apfs_invert command.  
We need do this process in standard installed macos system or inject apfs_invert to macos USB installer's BaseSystem.dmg.  

Restored APFS container has no VM volume, however don't be afraid.  
It will be automatically created in next boot process.  

### hint to next research...

It seems that overallocation problem (original container has) is fixed by inverting process.  
However, rebooted by restored container and use it, that problem will revive.  



(This research was written by HORIGUCHI Junshi, 2018/06/24.)
