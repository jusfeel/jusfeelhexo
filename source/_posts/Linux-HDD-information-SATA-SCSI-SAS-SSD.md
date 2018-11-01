---
title: Linux HDD information (SATA/SCSI/SAS/SSD)
date: 2018-10-28 23:19:38
tags:
---
  

![linux](https://www.reistlin.com/usr/uplinks/linux.gif "Linux HDD information (SATA/SCSI/SAS/SSD) - Jusfeel - FEEL.J.")

### **举例一：**

\[reistlin@reistlin.com ~\]$ cat /proc/scsi/scsi | grep Model   Vendor: ATA      Model: OCZ-VERTEX2 3.5  Rev: 1.27   Vendor: ATA      Model: OCZ-VERTEX2 3.5  Rev: 1.27

通过 Google 查询：OCZ-VERTEX2 3.5寸 固态硬盘（Firmware：1.27）

\[ [http://www.google.com.hk/search?q=**OCZ-VERTEX2+3.5**](http://www.google.com.hk/search?q=OCZ-VERTEX2+3.5) \]

### **举例二：**

\[reistlin@reistlin.com ~\]$ cat /proc/scsi/scsi | grep Model   Vendor: Slimtype Model: DVD A  DS8A5S    Rev: WC22   Vendor: ATA      Model: ST31000524NS     Rev: SN11   Vendor: ATA      Model: ST31000524NS     Rev: SN11   Vendor: ATA      Model: ST31000524NS     Rev: SN11

通过 Google 查询：ST31000524NS 希捷 SATA 1TB 7200转 32M

\[ [http://www.google.com.hk/search?q=**ST31000524NS**](http://www.google.com.hk/search?q=ST31000524NS) \]
s