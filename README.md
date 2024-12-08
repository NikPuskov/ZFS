# ZFS

root@ubuntu-jammy:/# lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

loop0    7:0    0  63.3M  1 loop /snap/core20/1879

loop1    7:1    0 111.9M  1 loop /snap/lxd/24322

loop2    7:2    0  53.2M  1 loop /snap/snapd/19122

sda      8:0    0    40G  0 disk

└─sda1   8:1    0    40G  0 part /

sdb      8:16   0    10M  0 disk

sdc      8:32   0   512M  0 disk

sdd      8:48   0   512M  0 disk

sde      8:64   0   512M  0 disk

sdf      8:80   0   512M  0 disk

sdg      8:96   0   512M  0 disk

sdh      8:112  0   512M  0 disk

sdi      8:128  0   512M  0 disk

sdj      8:144  0   512M  0 disk



root@ubuntu-jammy:/# zpool create nik1 mirror /dev/sdc /dev/sdd

root@ubuntu-jammy:/# zpool create nik2 mirror /dev/sde /dev/sdf

root@ubuntu-jammy:/# zpool create nik3 mirror /dev/sdg /dev/sdh

root@ubuntu-jammy:/# zpool create nik4 mirror /dev/sdi /dev/sdj

