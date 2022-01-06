# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

```bash
vagrant@vagrant:~$ truncate -s10G /tmp/sparse-file
vagrant@vagrant:~$ ls -lh !$
ls -lh /tmp/sparse-file
-rw-rw-r-- 1 vagrant vagrant 10G янв  6 01:44 /tmp/sparse-file
vagrant@vagrant:~$ du -sh !$
du -sh /tmp/sparse-file
0	/tmp/sparse-file
vagrant@vagrant:~$ cp /tmp/sparse-file /tmp/sparse-file1
vagrant@vagrant:~$ scp /tmp/sparse-file vagrant@localhost:/tmp/sparse_fi
vagrant@localhost's password: 
sparse-file                                                                                                                                                               100%   10GB 305.9MB/s   00:33    
vagrant@vagrant:~$ ls -lh /tmp/sparse_fi 
-rw-rw-r-- 1 dmitriy dmitriy 10G янв  6 01:46 /tmp/sparse_fi
vagrant@vagrant:~$ du -sh /tmp/sparse*
11G	/tmp/sparse_fi
0	/tmp/sparse-file
0	/tmp/sparse-file1
```
Провел мини исследование. Создал разреженный файл ls указывает его размер, но на диске он места не занимает, об этом говорит du. Скопировал файл cp, она поддерживает sparse, поэтому файл занимает тоже 0 МБ. Затем скопировал файл scp, и по времени копирования и по месту, занятому файлом видно, что scp не поддерживает sparse. 
Где-то пишут, что scp поддерживает параметр sparse=always, у себя не нашел. Для передачи sparse-файлов на удаленный компьютер можно использовать rsync или конвейер из tar и ssh
```bash
vagrant@vagrant:~$ tar -Szcf - /tmp/sparse-file | ssh vagrant@localhost 'mkdir /tmp/sparse/ && tar -C /tmp/sparse -zvxf -'
```

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Ответ: Нет. Права доступа ко всем жестким ссылкам на файл одинаковы, так как определяются одним и тем же индексным дескриптором.

```bash
vagrant@vagrant:~$ ls -lh /tmp/sparse_file_hard_link 
-rw-rw-r-- 2 vagrant vagrant 10G Jan  5 23:12 /tmp/sparse_file_hard_link
vagrant@vagrant:~$ sudo chmod o+x /tmp/sparse_file_hard_link 
vagrant@vagrant:~$ ls -lh /tmp/sparse_file_hard_link 
-rw-rw-r-x 2 vagrant vagrant 10G Jan  5 23:12 /tmp/sparse_file_hard_link
vagrant@vagrant:~$ ls -lh /tmp/sparse_file*
-rw-rw-r-x 2 vagrant vagrant 10G Jan  5 23:12 /tmp/sparse_file
-rw-rw-r-x 2 vagrant vagrant 10G Jan  5 23:12 /tmp/sparse_file_hard_link
vagrant@vagrant:~$ sudo chown root /tmp/sparse_file_hard_link 
vagrant@vagrant:~$ ls -lh /tmp/sparse_file*
-rw-rw-r-x 2 root vagrant 10G Jan  5 23:12 /tmp/sparse_file
-rw-rw-r-x 2 root vagrant 10G Jan  5 23:12 /tmp/sparse_file_hard_link
```

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
```bash
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk 
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part 
└─sda5                 8:5    0 63.5G  0 part 
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk 
sdc                    8:32   0  2.5G  0 disk 
```
4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

```bash
vagrant@vagrant:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xee8cc381.

Command (m for help): p
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xee8cc381

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-5242879, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G     

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (4196352-5242879, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): p
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xee8cc381

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

```bash
agrant@vagrant:~$ sudo sfdisk -d /dev/sdb |  sudo sfdisk -f /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x835a021b.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x835a021b

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
vagrant@vagrant:~$ sudo mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdb2                 8:18   0  511M  0 part  
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdc2                 8:34   0  511M  0 part
```

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
```bash
vagrant@vagrant:~$ sudo mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdb2                 8:18   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdc2                 8:34   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
```
Запишем конфигурацию RAID в файл

```bash
vagrant@vagrant:~$ echo -e "DEVICE partitions \n$(sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}')" | sudo tee /etc/mdadm/mdadm.conf 
DEVICE partitions 
ARRAY /dev/md0 level=raid1 num-devices=2 metadata=1.2 name=vagrant:0 UUID=5fe7cf2a:25cd02f8:e6655b15:a1e136c2
ARRAY /dev/md1 level=raid0 num-devices=2 metadata=1.2 name=vagrant:1 UUID=104b83ab:a81a3bcf:5f217552:5d9bead0
```
8. Создайте 2 независимых PV на получившихся md-устройствах.

```bash
vagrant@vagrant:~$ sudo pvcreate /dev/md0 /dev/md1 
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
vagrant@vagrant:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               vgvagrant
  PV Size               <63.50 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              16255
  Free PE               0
  Allocated PE          16255
  PV UUID               Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z
   
  "/dev/md0" is a new physical volume of "<2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/md0
  VG Name               
  PV Size               <2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               nrNryd-eDR4-7B8s-KK93-cWeE-I2nH-Mi6HmY
   
  "/dev/md1" is a new physical volume of "1018.00 MiB"
  --- NEW Physical volume ---
  PV Name               /dev/md1
  VG Name               
  PV Size               1018.00 MiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               arxl9x-BIC6-Qcue-XbJj-Q8tL-mi9l-Y87AtZ
  ```

9. Создайте общую volume-group на этих двух PV.

```bash
vagrant@vagrant:~$ vgcreate vg01 /dev/md0 /dev/md1
  WARNING: Running as a non-root user. Functionality may be unavailable.
  /run/lock/lvm/P_global:aux: open failed: Permission denied
vagrant@vagrant:~$ sudo vgcreate vg01 /dev/md0 /dev/md1
  Volume group "vg01" successfully created
vagrant@vagrant:~$ sudo vgdisplay 
  --- Volume group ---
  VG Name               vgvagrant
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <63.50 GiB
  PE Size               4.00 MiB
  Total PE              16255
  Alloc PE / Size       16255 / <63.50 GiB
  Free  PE / Size       0 / 0   
  VG UUID               PaBfZ0-3I0c-iIdl-uXKt-JL4K-f4tT-kzfcyE
   
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0   
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               fKWjS3-G04w-WntE-CaO8-ymlM-XwCC-bBdXlB
```
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
vagrant@vagrant:~$ sudo lvcreate -L100M -n lv01 vg01 /dev/md1 
  Logical volume "lv01" created.
vagrant@vagrant:~$ sudo lvs
  LV     VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv01   vg01      -wi-a----- 100.00m                                                    
  root   vgvagrant -wi-ao---- <62.54g                                                    
  swap_1 vgvagrant -wi-ao---- 980.00m 
  ```

11. Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
vagrant@vagrant:~$ sudo mkfs.ext4 /dev/vg01/lv01 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdb2                 8:18   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
    └─vg01-lv01      253:2    0  100M  0 lvm   
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdc2                 8:34   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
    └─vg01-lv01      253:2    0  100M  0 lvm
```

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
vagrant@vagrant:~$ mkdir /tmp/new
vagrant@vagrant:~$ mount /dev/vg
vg01/        vga_arbiter  vgvagrant/   
vagrant@vagrant:~$ mount /dev/vg01/lv01 /tmp/new/
mount: only root can do that
vagrant@vagrant:~$ sudo mount /dev/vg01/lv01 /tmp/new/
vagrant@vagrant:~$ mount | grep lv01
/dev/mapper/vg01-lv01 on /tmp/new type ext4 (rw,relatime,stripe=256)
```
13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```bash
vagrant@vagrant:~$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2022-01-06 01:50:05--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 21619846 (21M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz                                   100%[================================================================================================================>]  20.62M  5.48MB/s    in 3.5s    

2022-01-06 01:50:08 (5.97 MB/s) - ‘/tmp/new/test.gz’ saved [21619846/21619846]
```
14. Прикрепите вывод `lsblk`.
```bash
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdb2                 8:18   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
    └─vg01-lv01      253:2    0  100M  0 lvm   /tmp/new
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdc2                 8:34   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
    └─vg01-lv01      253:2    0  100M  0 lvm   /tmp/new
```
15. Протестируйте целостность файла:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
```bash
vagrant@vagrant:~$ gzip -t /tmp/new/test.gz; echo $?
0
```
16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
```bash
vagrant@vagrant:~$ sudo pvmove -n lv01 /dev/md1 /dev/md0
  /dev/md1: Moved: 16.00%
  /dev/md1: Moved: 100.00%
vagrant@vagrant:~$ lsblk 
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
│   └─vg01-lv01      253:2    0  100M  0 lvm   /tmp/new
└─sdb2                 8:18   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0 
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
│   └─vg01-lv01      253:2    0  100M  0 lvm   /tmp/new
└─sdc2                 8:34   0  511M  0 part  
  └─md1                9:1    0 1018M  0 raid0
```
17. Сделайте `--fail` на устройство в вашем RAID1 md.
```bash
agrant@vagrant:~$ sudo mdadm --fail /dev/md0 /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
```
18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
```bash
agrant@vagrant:~$ dmesg | tail -20
[ 3176.594840]  sdc: sdc1 sdc2
[ 3176.878012]  sdc: sdc1 sdc2
[ 3176.886261]  sdc: sdc1 sdc2
[ 3491.751057]  sdc: sdc1 sdc2
[ 3560.636642]  sdc: sdc1 sdc2
[ 3560.922766]  sdc: sdc1 sdc2
[ 3560.937993]  sdc: sdc1 sdc2
[ 3625.845074]  sdc: sdc1 sdc2
[ 3626.145532]  sdc: sdc1 sdc2
[ 3626.157987]  sdc: sdc1 sdc2
[ 5230.198266] md/raid1:md0: not clean -- starting background reconstruction
[ 5230.198268] md/raid1:md0: active with 2 out of 2 mirrors
[ 5230.198284] md0: detected capacity change from 0 to 2144337920
[ 5230.199280] md: resync of RAID array md0
[ 5240.399954] md: md0: resync done.
[ 5360.175206] md1: detected capacity change from 0 to 1067450368
[ 6968.904421] EXT4-fs (dm-2): mounted filesystem with ordered data mode. Opts: (null)
[ 6968.904431] ext4 filesystem being mounted at /tmp/new supports timestamps until 2038 (0x7fffffff)
[ 7943.039316] md/raid1:md0: Disk failure on sdb1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```
19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
```bash
vagrant@vagrant:~$ gzip -t /tmp/new/test.gz; echo $?
0
```
20. Погасите тестовый хост, `vagrant destroy`.
