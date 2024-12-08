# ZFS

1. Определение алгоритма  с наилучшим сжатием

Список всех дисков в виртуалке:

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

Создание 4х пулов из 2х дисков в RAID1:

root@ubuntu-jammy:/# zpool create nik1 mirror /dev/sdc /dev/sdd

root@ubuntu-jammy:/# zpool create nik2 mirror /dev/sde /dev/sdf

root@ubuntu-jammy:/# zpool create nik3 mirror /dev/sdg /dev/sdh

root@ubuntu-jammy:/# zpool create nik4 mirror /dev/sdi /dev/sdj

Информация о пулах:

root@ubuntu-jammy:/# zpool list

NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT

nik1   480M   105K   480M        -         -     0%     0%  1.00x    ONLINE  -

nik2   480M   105K   480M        -         -     0%     0%  1.00x    ONLINE  -

nik3   480M   100K   480M        -         -     0%     0%  1.00x    ONLINE  -

nik4   480M   100K   480M        -         -     0%     0%  1.00x    ONLINE  -

Добавление разных алгоритмов сжатия в каждую файловую систему:

root@ubuntu-jammy:/# zfs set compression=lzjb nik1

root@ubuntu-jammy:/# zfs set compression=lz4 nik2

root@ubuntu-jammy:/# zfs set compression=gzip-9 nik3

root@ubuntu-jammy:/# zfs set compression=zle nik4

Файловые системы имеют разные методы сжатия:

root@ubuntu-jammy:/# zfs get all | grep compression

nik1  compression           lzjb                   local

nik2  compression           lz4                    local

nik3  compression           gzip-9                 local

nik4  compression           zle                    local

Скачиваем один и тот же файл во все пулы:

root@ubuntu-jammy:/# for i in {1..4}; do wget -P /nik$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

Проверим:

root@ubuntu-jammy:/# ls -l /nik*

/nik1:

total 22092

-rw-r--r-- 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/nik2:

total 18004

-rw-r--r-- 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/nik3:

total 10965

-rw-r--r-- 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

/nik4:

total 40173

-rw-r--r-- 1 root root 41107603 Dec  2 08:56 pg2600.converter.log

Самый оптимальный метод сжатия в пуле nik3

root@ubuntu-jammy:/# zfs list

NAME   USED  AVAIL     REFER  MOUNTPOINT

nik1  21.7M   330M     21.6M  /nik1

nik2  17.7M   334M     17.6M  /nik2

nik3  10.8M   341M     10.7M  /nik3

nik4  39.4M   313M     39.3M  /nik4

root@ubuntu-jammy:/# zfs get all | grep compressratio | grep -v ref

nik1  compressratio         1.81x                  -

nik2  compressratio         2.23x                  -

nik3  compressratio         3.65x                  -

nik4  compressratio         1.00x                  -

Вывод: gzip-9 самый оптимальный алгоритм сжатия

2. Определение настроек пула

Скчиваем файл:

root@ubuntu-jammy:/# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'

Разархивируем его:

root@ubuntu-jammy:/# tar -xzvf archive.tar.gz

zpoolexport/

zpoolexport/filea

zpoolexport/fileb

root@ubuntu-jammy:/# zpool import -d zpoolexport/

   pool: otus

     id: 6554193320433390805

  state: ONLINE

status: Some supported features are not enabled on the pool.

        (Note that they may be intentionally disabled if the

        'compatibility' property is set.)

 action: The pool can be imported using its name or numeric identifier, though

        some features will not be available without an explicit 'zpool upgrade'.

 config:

        otus                    ONLINE
          mirror-0              ONLINE
            /zpoolexport/filea  ONLINE
            /zpoolexport/fileb  ONLINE

root@ubuntu-jammy:/# zpool import -d zpoolexport/ otus
