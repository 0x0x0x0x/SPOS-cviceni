# Cviceni 2

## Ziskani informaci o discich a souborovych systemech

```bash
cat /proc/partitions
cat /proc/mdstat
# velikosti fs
df -h | df -m | df -i
du -h | du -hs | du -m
mount
#připojí všechno z fstab
mount -a
mount /dev/sda1 /mnt/
fdisk -l /dev/sda
cfdisk /dev/sda
sfdisk -s
sfdisk -d /dev/sda | sfdisk /dev/sdb
/etc/fstab
lsmod
```

## Zakladni operace s FS

```bash
mkfs.ext4 /dev/sda1
mkfs.xfs /dev/sda2
mount /dev/sda2 /mnt/

# resize fs na velikost oblasti
resize2fs /dev/sda1
xfs_grow /dev/sda2

mkswap /dev/sda3
swapon /dev/sda3
```

## Vytvoření virtuálního disku
```bash
# vytvoří nový virtuální disk, pokud disk neexistuje
cfdisk /dev/vdb
# vytvoří filesystem
mkfs.ext4 /dev/vdb1
# přidáme virtuální disk na seznam připojitelných souborových systémů
nano /etc/fstab
	/dev/vdb1	/mnt	ext4	defaults	0	0
# připojíme virtuální disk
mount /dev/vdb1

# přepíše tabulku oblastí nulami (rychlé, ale nebezpečné)
dd if=/dev/zero of=/dev/vdb bs=1M count=1
```

## Random věci z cvičení
- ani cvičící nerozumí tomu, co vyváděl
- nějaká magie na resize, která asi ani nic nedělá
- lepší je reboot
```bash
kpartx -usf /dev/vdb
kpartx -a /dev/vdb
kpartx -l /dev/vdb
kpartx -d /dev/vdb
lsblk
```

## Vytvoření 1000 souborů
```bash
for i in $(seq 1 1000); do touch $i; done
```

## Prace s obrazy

```bash
# Kopírování /vdb do /vdc
dd if=/dev/vdb of=/dev/vdc

# Vyrobení 1 gb disk image
dd if=/dev/zero of=./disk.img bs=1M count=1024
# image jako block device
losetup -f ./disk.img
ls -l /dev/loop0
mkfs.ext4 /dev/loop0
mount  /dev/loop0 ./loop

#Mountuje adresáře mezi sebou
mount -o bind /mnt/ /root/mnt
```

## LVM

```bash
# Instalace
apt install lvm2
# vytvoří fyzickou oblast
pvcreate /dev/vdb
# ukáže fyzické oblasti
pvs
# vytvoří volume group
vgcreate data /dev/vdb /dev/vdc /dev/vdd
# ukáže volume skupiny
vgs
# vytvoří LVM mysql pro skupinu data
lvcreate -n mysql -L 2.5G data
# listne LVM
lvs
# formátování
mkfs.ext4 /dev/data/mysql
# změna velikosti LVM
lvresize -L +100M /dev/data/mysql
resize2fs /dev/data/mysql

umount /dev/data/mysql
lvresize -L -110M /dev/data/mysql
fsck.ext4 /dev/data/mysql

# odstranění LVM
lvremove /dev/data/post-gres

# zvětšení swapu
lvcreate -n swap -L 100M data
mkswap /dev/data/swap
swapon /dev/data/swap
# swapon / swapoff používá image jako swap

#mažeme všechno
dd if=/dev/zero of=/dev/vdb bs=1M count=1
# ukazuje, že už nic nemáme
lvs
vgs
pvs
```

```bash
pvdisplay | pvscan | pvremove 
pvcreate /dev/md0
pvmove -v /dev/sda2 /dev/sdb2

vgdisplay | vgscan | vgchange | vgremove | 
vgcreate vg-data/dev/md0
vgextend vg-data /dev/md1
vgreduce vg-data /dev/md0

lvdisplay | lvscan | lvchange | lvresize | lvrename
lvcreate -L 30G -n db-data vg-data
lvextend -L 60G -n db-data vg-data 
resize2fs /dev/vg-data/db-data
lvcreate -L 30G -m1 -n db-data vg-data
lvcreate --size 100M --snapshot -n db-data-snap /dev/vg-data/db-data
lvremove /dev/vg-data/db-data-snap

mount /dev/vg-data/db-data /mnt/data1
mount /dev/vg-data/db-data-snap /mnt/data2

vgcfgbackup
vgcfgrestore
```


## RAID

```
raid0 - linear / striping
raid1 - mirroring
raid5 - striping + parita
raid6 - striping + 2x parita

raid01 - raid0 + raid1
raid10 - raid1 + raid0
raid50 - raid5 + raid0
raid60 - raid6 + raid0
```

### Sprava RAIDu

```bash
# 
apt install mdadm
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 \
	/dev/sda1 /dev/sdb1

# zrcadlení mezi dvěma virtuálními disky
mdadm -Cv /dev/md0 -l1 -n2 /dev/vd[bc]1
mdadm -Cv /dev/md1 -l5 -n3 /dev/vd[bc]2 missing

pvcreate /dev/md0
pvcreate /dev/md1
vgcreate data /dev/md[01]
lvcreate -L 1G -n mysql data

dd if=/dev/zero of=/loop-520MB bs=1M count=530
losetup -f /loop-520MB
mdadm --add /dev/md1 /dev/loop0

mdadm --remove /dev/md1 /dev/loop0
mdadm --fail /dev/md1 /dev/vdb
# zotavení
lvchange -a n /dev/data/mysql
mdadm --stop /dev/md1
mdadm -A /dev/md1 /dev/vdb2 /dev/vdc2 /dev/loop0
lvchange -a y /dev/data/mysql




mdadm -Cv /dev/md1 -l5  -n5 /dev/sda1 /dev/sdb1 /dev/sdc1 \
	/dev/sdd1 /dev/sde1

mdadm -Cv /dev/md1 -l5  -n4 -x1 /dev/sda1 /dev/sdb1 /dev/sdc1 \
	/dev/sdd1 /dev/sde1

mdadm -Cv /dev/md1 -l5  -n3 /dev/sda1 /dev/sdb1 missing

mdadm --fail /dev/md0 /dev/sda1
mdadm --remove /dev/md0 /dev/sda1
mdadm --add /dev/md0 /dev/sdc1

mdadm /dev/md0 --fail /dev/sda1 --remove /dev/sda1 --add /dev/sdc1
 # přidá další disk k poli
 mdadm --add /dev/md0 /dev/sdc1
 # rozroste pole na počet disků
 mdadm --grow /dev/md0 -n3

# aktivní raidy
cat /proc/mdstat
mdadm --detail /dev/md0

watch --interval=10 cat /proc/mdstat

lvremove /dev/data/mysql
vgremove data
pvremove /dev/md0

mdadm --stop /dev/md0
mdadm --remove /dev/md0

mdadm --zero-superblock /dev/vdb1

# odstranění loop
losetup -l
losetup -d /dev/loop0
rm /loop-520MB

```
