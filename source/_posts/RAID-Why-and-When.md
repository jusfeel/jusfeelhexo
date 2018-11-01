---
title: RAID Why and When
date: 2018-10-28 22:22:32
tags:
---

RAID stands for Redundant Array of Independent Disks (some are taught "Inexpensive" to indicate that they are "normal" disks; historically there were internally redundant disks which were very expensive; since those are no longer available the acronym has adapted).

At the most general level, a RAID is a group of disks that act on the same reads and writes. SCSI IO is performed on a volume ("LUN"), and these are distributed to the underlying disks in a way that introduces a performance increase and/or a redundancy increase. The performance increase is a function of striping: data is spread across multiple disks to allow reads and writes to use all the disks' IO queues simultaneously. Redundancy is a function of mirroring. Entire disks can be kept as copies, or individual stripes can be written multiple times. Alternatively, in some types of raid, instead of copying data bit for bit, redundancy is gained by creating special stripes that contain parity information, which can be used to recreate any lost data in the event of a hardware failure.

There are several configurations that provide different levels of these benefits, which are covered here, and each one has a bias toward performance, or redundancy.

An important aspect in evaluating which RAID level will work for you depends on it's advantages and hardware requirements (E.g.: number of drives).

Another important aspect of _most_ of these types of RAID (0,1,5) is that they do _not_ ensure the integrity of your data, because they are abstracted away from the actual data being stored. So RAID does not protect against corrupted files. If a file is corrupted by _any_ means, the corruption will be mirrored or paritied and committed to the disk regardless. However, [RAID-Z does claim to provide file-level integrity of your data](http://www.freebsd.org/doc/handbook/filesystems-zfs.html).

* * *

Direct attached RAID: Software and Hardware
===========================================

There are two layers at which RAID can be implemented on direct attached storage: hardware and software. In true hardware RAID solutions, there is a dedicated hardware controller with a processor dedicated to RAID calculations and processing. It also typically has a battery-backed cache module so that data can be written to disk, even after a power failure. This helps to eliminate inconsistencies when systems are not shut down cleanly. Generally speaking, good hardware controllers are better performers than their software counterparts, but they also have a substantial cost and increase complexity.

Software RAID typically does not require a controller, since it doesn't use a dedicated RAID processor or a separate cache. Typically these operations are handled directly by the CPU. In modern systems, these calculations consume minimal resources, though some marginal latency is incurred. RAID is handled by either the OS directly, or by a faux controller in the case of [FakeRAID](http://serverfault.com/q/9244/10472).

Generally speaking, if someone is going to choose software RAID, they should avoid FakeRAID and use the OS-native package for their system such as Dynamic Disks in Windows, mdadm/LVM in Linux, or ZFS in Solaris, FreeBSD, and other related distributions. FakeRAID use a combination of hardware and software which results in the initial appearance of hardware RAID, but the actual performance of software RAID. Additionally it is commonly extremely difficult to move the array to another adapter (should the original fail).

* * *

Centralized Storage
===================

The other place RAID is common is on centralized storage devices, usually called a SAN (Storage Area Network) or a NAS (Network Attached Storage). These devices manage their own storage and allow attached servers to access the storage in various fashions. Since multiple workloads are contained on the same few disks, having a high level of redundancy is generally desirable.

The main difference between a NAS and a SAN is block vs. file system level exports. A SAN exports a whole "block device" such as a partition or logical volume (Including those built on top of a RAID array.). Examples of SAN's include Fibre Channel and iSCSI. A NAS exports a "file system" such as a file or folder. Examples of NAS's include CIFS/SMB (Windows file sharing) and NFS.

* * *

RAID 0
======

### Good when: Speed at all costs

### Bad when: You care about your data

RAID0 (aka Striping) is sometimes referred to as "the amount of data you will have left when a drive fails". It really runs against the grain of "RAID", where the "R" stands for "Redundant".

RAID0 takes your block of data, splits it up into as many pieces as you have disks (2 disks → 2 pieces, 3 disks → 3 pieces) and then writes each piece of the data to a seperate disk.

This means that a single disk failure destroys the entire array (because you have Part 1 and Part 2, but no Part 3), but it provides very fast disk access.

It is not often used in production environments, but it could be used in a situation where you have strictly temporary data that can be lost without repercussions. It is used somewhat commonly for caching devices (such as an L2Arc device).

The total usable disk space is the sum of all the disks in the array added together (e.g. 3x 1TB disks = 3TB of space)

![RAID 1](https://i.imgur.com/l5E1Y.png "RAID - Jusfeel - FEEL.J.")

* * *

RAID 1
======

### Good when: You have limited number of disks but need redundancy

### Bad when: You need a lot of storage space

RAID 1 (aka Mirroring) takes your data and duplicates it identically on two or more disks (although typically only 2 disks). It is the only way to ensure data redundancy when you have less than three disks.

RAID 1 sometimes improves read performance. Some implementations of RAID 1 will read from both disks to double the read speed. Some will only read from one of the disks, which does not provide any additional speed advantages. Others will read the same data from both disks, ensuring the array's integrity on every read, but this will result in the same read speed as a single disk.

It is typically used in small servers that have very little disk expansion, such as 1RU servers that may only have space for two disks or in workstations that require redundancy. Because of its high overhead of "lost" space, it can be cost prohibitive with small-capacity, high-speed (and high-cost) drives, as you need to spend twice as much money to get the same level of usable storage.

The total usable disk space is the size of the smallest disk in the array (e.g. 2x 1TB disks = 1TB of space).

![RAID 1](https://i.imgur.com/4CTnh.png "RAID - Jusfeel - FEEL.J.")

* * *

RAID 1E
-------

The _1E_ RAID level is similar to RAID 1 - data is always written to (at least) two disks. But unlike RAID1, it allows for an odd number of disks by simply interleaving the data blocks among several disks.

Performance characteristics are similar to RAID1, fault tolerance is similar to RAID 10.

![RAID 1E](https://i.imgur.com/OSYTS.png "RAID - Jusfeel - FEEL.J.")

* * *

RAID 10
=======

### Good when: You want speed and redundancy

### Bad when: You can't afford to lose half your disk space

RAID 10 is a combination of RAID 1 and RAID 0. The order of the 1 and 0 is very important. Say you have 8 disks, it will create 4 RAID 1 arrays, and then apply a RAID 0 array on top of the 4 RAID 1 arrays.

This means that one disk from each pair can fail. So if you have sets A, B, C and D with disks A1, A2, B1, B2, C1, C2, D1, D2, you can lose one disk from each set (A,B,C or D) and still have a functioning array.

However, if you lose two disks from the same set, then the array is totally lost. You can lose _up to_ (but not guaranteed) 50% of the disks.

You are guaranteed high speed and high availability in RAID 10.

RAID 10 is a very common RAID level, especially with high capacity drives where a single disk failure makes a second disk failure more likely before the RAID array is rebuilt. During recovery, the performance degradation is much lower than it's RAID 5 counterpart as it only has to read from one drive to reconstruct the data.

The avaliable disk space is 50% of the sum of the total space. (e.g. 8x 1TB drives = 4TB of usable space).

![RAID 10](https://i.imgur.com/KmLda.png "RAID - Jusfeel - FEEL.J.")

* * *

RAID 01
=======

### Good when: _never_

### Bad when: _always_

It is the reverse of RAID 10. It creates two RAID 0 arrays, and then puts a RAID 1 over the top. This means that you can lose one disk from each set (A1, A2, A3, A4 or B1, B2, B3, B4).

To be absolutely clear:

*   If you have a RAID10 array with 8 disks and one dies (we'll call it A1) then you'll have 6 redundant disks and 1 without redundancy. If another disk dies there's a **85%** chance your array is still working.
*   If you have a RAID01 array with 8 disks and one dies (we'll call it A1) then you'll have 4 redundant disks and 3 without redundancy. If another disk dies there's a **57%** chance your array is still working.

It provides no additional speed over RAID 10, but substantially less redundancy and should be avoided at all costs.

* * *

RAID 5
======

### Good when: You want a balance of redundancy and disk space or have a large sequential write workload.

### Bad when: You have a high random write workload or large drives.

RAID 5 uses a simple XOR operation to calculate parity. Upon single drive failure, the information can be reconstructed from the remaining drives using the XOR operation on the known data.

However, this rebuilding process is very intensive and array performance can suffer heavily during the rebuild procedure. Given the lengthy amount of time it takes to perform this rebuild (days!) it is not usually recommended to use RAID5 on large drives, as the risk of data loss during a rebuild is increased relative to disk size. Because every sector on every disk is needed to calculate the data to be rebuilt on the replaced disk, there is an increased risk of data loss from an already present but previously undetected error, or a new failure from the stress of the rebuild, on one of those disks - either in the form of an unrecoverable read error, or silent data corruption (bit flipping).

Another caveat is the "write hole," which describes a condition where a failure of some kind (power, system, RAID controller, etc.) occurs mid-write, with new data being written to some, but not all, of a parity stripe. In this situation, it's not clear to the device what data can be trusted and what cannot - the entire stripe is rendered corrupt. Battery-backed write caches on RAID controllers the main mitigation technique for this issue.

RAID 5 has a high disk write overhead on small writes. Writes less than a stripe width in size cause an extra read prior to the write, causing a single frontend IOP to turn into 4 backend IOPs. The small write penalty is mitigated by having controller-based write-back caches capable of taking up the entire I/O write load of your system. However, once the cache fills up (or _crams_), all activity on the array will cease until enough data has been synced do the disks. RAID5 can also give better performance than RAID10 for large sequential writes since it has more spindles to distribute writes onto. The computational overhead of the parity calculation is hardly a concern nowadays - hardware I/O processors used in more recent RAID controllers are capable of calculating parity at full wirespeed of the interface.

However, RAID 5 is the most cost effective solution of adding redundant storage to an array, as it requires the loss of only 1 disk (E.g. 4x 1TB disks = 3TB of usable space). It requires a minimum of 3 disks.

There has been a lot of controversy about striping with parity in the past, many people regard it not being worth the money if you need high array performance in terms of IOPS - which is a typical requirement for database applications or SAN systems used for virtualization. The [_Battle Against Any RAID Five_ (BAARF) initiative website](http://www.miracleas.com/BAARF/) provides a lot of information about the internal workings of striping with parity and its caveats.

![RAID 5](https://i.imgur.com/oHoJc.png "RAID - Jusfeel - FEEL.J.")

* * *

RAID 6
======

### Good when: You want a balance of redundancy and disk space or have a large sequential write workload.

### Bad when: You have a high random write workload.

RAID 6 is similar to RAID 5 but it uses two disks worth of parity instead of just one (the first is XOR, the second is a LSFR), so you can lose two disks from the array with no data loss. The write penalty is higher than RAID 5 and you have one less disk of space, so this option is best geared towards arrays that do a lot of reads or large sequential writes and when RAID 10 isn't an option because of capacity.

* * *

RAID 50
=======

### Good when: You have _a lot_ of disks that need to be in a single array and RAID 10 isn't an option because of capacity.

### Bad when: You have so many disks that many simultaneous failures are possible before rebuilds complete. Or when you don't have many disks.

RAID 50 is a nested level, much like RAID 10. It combines two or more RAID 5 arrays and stripes data across them in a RAID 0. This offers both performance and multiple disk redundancy, as long as multiple disks are lost from **different** RAID 5 arrays.

In a RAID 50, disk capacity is n-x, where x is the number of RAID 5s that are striped across. For example, if a simple 6 disk RAID 50, the smallest possible, if you had 6x1TB disks in two RAID 5s that were then striped across to become a RAID 50, you would have 4TB usable storage.

* * *

RAID 60
=======

### Good when: You have a similar use case to RAID 50, but need more redundancy.

### Bad when: You don't have a substantial number of disks in the array.

RAID 6 is to RAID 60 as RAID 5 is to RAID 50. Essentially, you have more than one RAID 6 that data is then striped across in a RAID 0. This setup allows for up to two members of any individual RAID 6 in the set to fail without data loss. Rebuild times for RAID 60 arrays can be substantial, so it's usually a good idea to have one hot-spare for each RAID 6 member in the array.

In a RAID 60, disk capacity is n-2x, where x is the number of RAID 6s that are striped across. For example, if a simple 8 disk RAID 60, the smallest possible, if you had 8x1TB disks in two RAID 6s that were then striped across to become a RAID 60, you would have 4TB usable storage. As you can see, this gives the same amount of usable storage that a RAID 10 would give on an 8 member array. While RAID 60 would be slightly more redundant, the rebuild times would be substantially larger. Generally, you want to consider RAID 60 only if you have a large number of disks.

* * *

RAID-Z
======

### Good when: You are using ZFS on a system that supports it.

### Bad when: Performance demands hardware RAID acceleration.

RAID-Z is a bit complicated to explain since ZFS radically changes how storage and file systems interact. ZFS encompasses the traditional roles of volume management (RAID is a function of a Volume Manager) and file system. Because of this, ZFS can do RAID at the file's storage block level rather than at the volume's strip level. This is exactly what RAID-Z does, write the file's storage blocks across multiple physical drives including a parity block for each set of stripes.

An example may make this much more clear. Say you have 3 disks in a ZFS RAID-Z pool, the block size is 4KB. Now you write a file to the system that is exactly 16KB. ZFS will split that into four 4KB blocks (as would a normal operating system); then it will calculate two blocks of parity. Those six blocks will be placed on the drives similar to how RAID-5 would distribute data and parity. This is an improvement over RAID5 in that there was no reading of existing data stripes to calculate the parity.

Another example builds on the previous. Say the file was only 4KB. ZFS will still have to build one parity block, but now the write load is reduced to 2 blocks. The third drive will be free to service any other concurrent requests. A similar effect will be seen anytime the file being written is not a multiple of the pool's block size multiplied by the number of drives less one (ie \[File Size\] <> \[Block Size\] * \[Drives - 1\]).

ZFS handling both Volume Management and File System also means you don't have to worry about aligning partitions or stripe-block sizes. ZFS handles all that automatically with the recommended configurations.

The nature of ZFS counteracts some of the classic RAID-5/6 caveats. All writes in ZFS are done in a copy-on-write fashion; all changed blocks in a write operation are written to a new location on disk, instead of overwriting the existing blocks. If a write fails for any reason, or the system fails mid-write, the write transaction either occurs completely after system recovery (with the help of the ZFS intent log) or does not occur at all, avoiding potential data corruption. Another issue with RAID-5/6 is potential data loss or silent data corruption during rebuilds; regular `zpool scrub` operations can help to catch data corruption or drive issues before they cause data loss, and checksumming of all data blocks will ensure that all corruption during a rebuild is caught.

The main disadvantage to RAID-Z is that it is still software raid (and suffers from the same minor latency incurred by the CPU calculating the write load instead of letting a hardware HBA offload it). This may be resolved in the future by HBA's that support ZFS hardware acceleration.

Origin:[http://serverfault.com/questions/339128/what-are-the-different-widely-used-raid-levels-and-when-should-i-consider-them](http://serverfault.com/questions/339128/what-are-the-different-widely-used-raid-levels-and-when-should-i-consider-them)