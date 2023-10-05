# Redundancia es discos (RAID)

fdisk

## Primer RAID 1

mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde

mdadm --create /dev/md2 --level=0 --raid-devices=2 /dev/md0 /dev/md1

mdadm --detail --scan >> /etc/mdadm.conf
