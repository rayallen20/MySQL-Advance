# 附录1 Ubuntu扩展磁盘

## STEP1. 查看新磁盘设备名

```
root@mysql-master:~# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.9M  1 loop /snap/core20/2105
loop1                       7:1    0  63.8M  1 loop /snap/core20/2599
loop2                       7:2    0 111.9M  1 loop /snap/lxd/24322
loop3                       7:3    0  40.4M  1 loop /snap/snapd/20671
loop4                       7:4    0  49.3M  1 loop /snap/snapd/24792
loop5                       7:5    0  89.4M  1 loop /snap/lxd/31333
sda                         8:0    0    20G  0 disk 
├─sda1                      8:1    0     1M  0 part 
├─sda2                      8:2    0   1.8G  0 part /boot
└─sda3                      8:3    0  18.2G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0  18.2G  0 lvm  /
sdb                         8:16   0    80G  0 disk 
```

```
root@mysql-master:~# fdisk -l
Disk /dev/loop0: 63.91 MiB, 67014656 bytes, 130888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 63.77 MiB, 66863104 bytes, 130592 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 111.95 MiB, 117387264 bytes, 229272 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop3: 40.43 MiB, 42393600 bytes, 82800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop4: 49.29 MiB, 51687424 bytes, 100952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop5: 89.4 MiB, 93745152 bytes, 183096 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: E30003FB-29AD-4C38-8E64-E2DF820C701C

Device       Start      End  Sectors  Size Type
/dev/sda1     2048     4095     2048    1M BIOS boot
/dev/sda2     4096  3719167  3715072  1.8G Linux filesystem
/dev/sda3  3719168 41940991 38221824 18.2G Linux filesystem


Disk /dev/sdb: 80 GiB, 85899345920 bytes, 167772160 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 18.22 GiB, 19566428160 bytes, 38215680 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

这里我的新设备名为`/dev/sdb`

## STEP2. 将新磁盘初始化为LVM物理卷

```
root@mysql-master:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

## STEP3. 将新物理卷加入到现有卷组

```
root@mysql-master:~# vgextend ubuntu-vg /dev/sdb
  Volume group "ubuntu-vg" successfully extended
```

## STEP4. 扩容根逻辑卷

```
root@mysql-master:~# lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 18.22 GiB (4665 extents) to <98.22 GiB (25144 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```

## STEP5. 扩展文件系统

```
root@mysql-master:~# resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 13
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 25747456 (4k) blocks long.
```

## STEP6. 检查空间

```
root@mysql-master:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              1.6G  1.6M  1.6G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   97G  7.1G   86G   8% /
tmpfs                              7.8G     0  7.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.8G  129M  1.5G   8% /boot
tmpfs                              1.6G  4.0K  1.6G   1% /run/user/0
```