---
layout: post
title: How to make array disk from  Unconfigured(bad) to Online
date: 2018-08-15 10:00:00.000000000 +08:00
tags:  Unconfigured(bad)  Online
---



## The disk in the array shows Unconfigured(bad)
```
[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64 -pdlist -aall  | grep ^Fir  
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Unconfigured(bad)
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
```
## Make the disk from Unconfigured(bad) to  Unconfigured(good) 
```
[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64  -PDMakeGood -PhysDrv[8:3] -a0 
                                     
Adapter: 0: EnclId-8 SlotId-3 state changed to Unconfigured-Good.

[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64 -pdlist -aall  | grep ^Fir     
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Unconfigured(good), Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
```
## check foreign configuration and clear it
```
[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64 -CfgForeign -Scan -a0 
                                     
There are 1 foreign configuration(s) on controller 0.

Exit Code: 0x00

[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64 -CfgForeign -Clear -a0 
                                     
Foreign configuration 0 is cleared on controller 0.

Exit Code: 0x00

[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64  -pdlist -aall | grep ^Fir
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Unconfigured(good), Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
```
## Set the disk to hot spares and will start rebuild
```
[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64  -PDHSP -Set  -PhysDrv[8:3] -a0 
                                     
Adapter: 0: Set Physical Drive at EnclId-8 SlotId-3 as Hot Spare Success.

Exit Code: 0x00
[root@kvm~]# /opt/MegaRAID/MegaCli/MegaCli64 -pdlist -aall  | grep ^Fir      
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Rebuild
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up

[root@kvm ~]# /opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -ShowProg -PhysDrv[8:3] -a0 
                                     
Rebuild Progress on Device at Enclosure 8, Slot 3 Completed 7% in 0 Minutes.

Exit Code: 0x00
```