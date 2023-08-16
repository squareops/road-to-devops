# Storage Management
In Linux, there are basically two methods a disk or drive can be partitioned. One way is by using the standard partitioning method, and the other is by using the LVM partitioning method.

## What Is Standard Partition In Linux
A standard partition is a method in Linux a drive/disk is splitted/partitioned.

Disks topology in Linux is in the form of (sda, sdb, sdc, sdn..), from first disk to the last disk respectively, and can be partitioned in a standard method as (sda1, sda2, sdb1, sdb2, sdn1, sdn2, etc).

```
[root@HQDEV1 ~]# lsblk

NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   15G  0 disk
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0   14G  0 part
  ├─rhel-root 253:0    0 12.5G  0 lvm  /
  └─rhel-swap 253:1    0  1.5G  0 lvm  [SWAP]
sdb             8:16   0    5G  0 disk
└─sdb1          8:17   0    1G  0 part /test1
sr0            11:0    1  7.3G  0 rom  /run/media/root/RHEL-8-1-0-BaseOS-x86_64

```
![](Images/storage.png)
From the screen-shot above, you can see that the disk, “sda” is partitioned to sda1 and sda2, while the “sdb” disk is partitioned to sdb1 only. It is very possible to use only one partitioning method which can be standard or LVM, or both partitioning methods. You will understand better as we move on in this subject matter.

The utilities that can be used to create a standard partition in Linux are “fdisk” and “gdisk”. We will see how these utilities can be used with examples in the “ACTION TIME” section.

Going forward, there are some important storage concepts we need to understand before we proceed. They are filesystem, mount point, and “/etc/fstab” file.

### Filesystem In Linux

A filesystem in this context is a way of arranging, or rather, a system of arranging files on a drive. Just as books can be arranged in a shelve horizontally or vertically, a filesystem is also a system files can be arranged on a device.

There are a lot of filesystems available in Linux. The most commonly used ones are; xfs, ext4, ext3, ext2, btrfs and gfs2. So, files can be arranged on a disk in any of these following systems.

The xfs filesystem is the default filesystem in RHEL 7 and above. Xfs has been considered to be a very good filesystem for a basic storage. It is flexible and has good tuning options.

The ext (2,3, and 4) were the default filesystems in RHEL6 and below. It uses a flat index file and not very scalable as compared to xfs, while btrfs is a copy-on-write filesystem. One advantage of using brtfs filesystem is to eradicate the need for journaling.

The gfs2 filesystem is used for active active clustering. For active passive type of clustering, any of the ext filesystems or the xfs filesystem can be used.

```
<script async="" src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle" style="display:block; text-align:center;" data-ad-layout="in-article" data-ad-format="fluid" data-ad-client="ca-pub-4045371711994399" data-ad-slot="6353597691"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
```

When a disk is partitioned, a filesystem has to be created on the partition before it can be used. In other words, a disk has to be partitioned with one particular filesystem.


To know the filesystem of a particular partition, use the command,
        blkid |grep <partition>

OR

        fsck -N <partition>

#### To see the type of filesystem on sdb1 for example, run the command,

```
[root@HQDEV1 ~]# blkid |grep /dev/sdb1

/dev/sdb1: LABEL="tekneed_fs" UUID="fe48facc-c481-4429-93d2-77e4ac0afc89" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="245ddd05-cdcb-4128-ba75-28eb026dcbaf"
[root@HQDEV1 ~]#
```
![](Images/storage1.png)

OR
```
[root@HQDEV1 ~]# fsck -N /dev/sdb1

fsck from util-linux 2.32.1
[/usr/sbin/fsck.ext4 (1) -- /dev/sdb1] fsck.ext4 /dev/sdb1
```

#### To create a filesystem, use the command,
        mkfs.<file-system type> /dev/<block or partition>

#### For example, to create the ext3 fileystem on /sdb1, use the command,

        [root@HQDEV1 ~]# mkfs.ext3 /dev/sdb1

In the “ACTION TIME”[link] section, we will see how all of these are done with examples.

#### To see more options of mkfs utility, please see the man page or use the command,

        mkfs.<fileystem-type> --help

example is 
        #mkfs.ext4 --help

Another important concept is mount point.

### Mount points In Linux
A mount point is a special directory or a path where drives or partitions are mounted on.

If you followed the “Learn Linux from scratch” series course on this site, where we discussed filesystem hierarchy and structure, you would know that the default mount point in Linux is “/mnt”.

You can mount any device on this directory if you don’t wish to create a mount point.

To see all the drives and partitions mounted on a Linux system, use the command,

```
[root@HQDEV1 ~]# df -hT

Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  886M     0  886M   0% /dev
tmpfs                 tmpfs     904M     0  904M   0% /dev/shm
tmpfs                 tmpfs     904M  9.7M  894M   2% /run
tmpfs                 tmpfs     904M     0  904M   0% /sys/fs/cgroup
/dev/mapper/rhel-root xfs        13G  4.4G  8.2G  35% /
/dev/sda1             xfs      1014M  180M  835M  18% /boot
tmpfs                 tmpfs     181M  1.2M  180M   1% /run/user/42
tmpfs                 tmpfs     181M  4.6M  177M   3% /run/user/0
/dev/sr0              iso9660   7.4G  7.4G     0 100% /run/media/root/RHEL-8-1-0-BaseOS-x86_64

```

From the output of the command above, you can see the drives and partitions(filesystems), its type, sizes, used and available sizes and mount points.

### How To Create A Mount Point In Linux
To create a mount point, use the “mkdir” command.

#### For example, to create a mount point “/tekneed/victor”, use the command,

```
[root@HQDEV1 ~]# mkdir /tekneed/victor
sometimes, you may want to use the “-p” option which means (no error if existing, make parent directories as needed)

[root@HQDEV1 ~]# mkdir -p /tekneed/victor
How To Manually Mount a disk or a partition In Linux
After a mount point has been created, the next thing is to mount the drive/partition or map a drive/partition to the mount point.
```

#### To manually mount a disk or partition, use the command,

        mount /dev/<drive-or-partition> <mount-point>

#### For example, to mount the partition, (sdb1) on the mount point, (/tekneed/victor), use the command,

        [root@HQDEV1 ~]# mount /dev/sdb1 /tekneed/victor

Sometimes, when a system reboots, a partition or a device number/topology may change. For example, “/dev/sdb1” may change to “/dev/sdc1”.

To avoid this and make the device numbers persistent after a reboot, always mount devices/partitions with their labels or UUIDs.

#### To mount a device/partition with a label, use the command,

        mount LABEL=<label-name> <mount-point>

We will see how all of these are done with examples in the “ACTION TIME” section.

#### How To Unmount a Device Or Partition In Linux

As an administrator, if you wish to unmount a device or a partition, use the command,

        umount /dev/<device-or-partition>

for example

        [root@HQDEV1 ~]# umount /dev/sdc2

#### To list all the disks with their partitions on a Linux system, use the command, “lsblk” (list blocks)

fdisk/partitioning, manage/extend filesystems - xfs/ext4, LVM, check filesystem and disk usage - df/du, /etc/fstab

i. [Linux filesystem](https://opensource.com/life/16/10/introduction-linux-filesystems)

ii. [Fdisk in linux](https://linuxhint.com/fdisk_in_linux/)

iii. [How to modify EBS volumes ?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modify-volume.html)

    a. Requirements when modifying volumes

    b. Request modifications to your EBS volumes

    c. Monitor the progress of volume modifications

    d. Extend a Linux file system after resizing a volume

### NOTE: Do not practice this section on your laptop or WSL, use an EC2 instance else you may corrupt your system
