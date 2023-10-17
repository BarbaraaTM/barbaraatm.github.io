# Redundancia es discos (RAID)

fdisk

## RAID 10

mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde

mdadm --create /dev/md2 --level=0 --raid-devices=2 /dev/md0 /dev/md1

mdadm --detail --scan > /etc/mdadm/mdadm.conf

mkfs.ext4 /dev/md0

mkdir /datos00

mount /dev/md0 /datos00

df -h

## Discos de repuesto

RAID 1 (sdb y sdc) = 126
RAID 1 (sdd y sde) = 127
RAID 0 = 125

mdadm --add /dev/md126 /dev/sdf

mdadm --add /dev/md127 /dev/sdg

## Prueba fichero modificado

dd if=/dev/urandom of=prueba.txt bs=500MB count=1

md5sum prueba.txt > checksum.txt

echo "Modificado" >> prueba.txt

md5sum -c checksum.txt

## Fallo disco

mdadm /dev/md126 --fail /dev/sdb --remove /dev/sdb

## LVM

pvcreate /dev/md125

vgcreate SeguridadAD /dev/md125

lvcreate -l 70%FREE -n P1 SeguridadAD

lvcreate -l 30%FREE -n P2 SeguridadAD

mkfs.ext4 /dev/SeguridadAD/P1

mkfs.ext4 /dev/SeguridadAD/P2

mkdir -p /datos001

mkdir -p /datos002

mount /dev/SeguridadAD/P1 /datos001

mount /dev/SeguridadAD/P2 /datos002


