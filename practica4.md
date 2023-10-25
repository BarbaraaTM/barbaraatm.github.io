# Redundancia es discos (RAID)

## Introducción

Vamos a crear un RAID 10, para ello usaremos 4 discos del mismo tamaño.

    ```bash
    $ fdisk
    ```

## Creación del RAID 10

- Paso 1

  Creamos dos RAID 1 con dos discos cada uno:
    ```bash
    $ mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
    $ mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
    ```

- Paso 2

  Creamos un RAID 0 con los dos RAID 1 que hemos creado antes:
   ```bash
    $ mdadm --create /dev/md2 --level=0 --raid-devices=2 /dev/md0 /dev/md1
    ```

- Paso 3

  Guardamos en el archivo /etc/mdadm/mdadm.conf la información del arreglo:
   ```bash
    $ mdadm --detail --scan > /etc/mdadm/mdadm.conf
    ```

- Paso 4

  Configuramos el RAID0 para poder usarlo como unidad de almacenamiento:
  ```bash
    $ mkfs.ext4 /dev/md0
    $ mkdir /datos00
    $ mount /dev/md0 /datos00
    $ df -h
    ```

## Discos de repuesto

Para realizar esta parte de la práctica, deberás añadir otros dos discos del mismo tamaño que los anteriores.

*Al reiniciar los nombres de los antiguos discos han cambiado a los siguientes:*

RAID 1 (sdb y sdc) = 126
RAID 1 (sdd y sde) = 127
RAID 0 = 125

---

- Paso 1

  Añadimos uno a cada RAID 1:
  ```bash
    $ mdadm --add /dev/md126 /dev/sdf
    $ mdadm --add /dev/md127 /dev/sdg
    ```

## Fallo disco

Vamos a similar el fallo de uno de los discos de cada RAID 1 para comprobar que funciona el sistema de discos de repuesto.

- Paso 1

  Simulamos el error del disco:
   ```bash
    $ mdadm /dev/md126 --fail /dev/sdb --remove /dev/sdb
    $ mdadm /dev/md127 --fail /dev/sde --remove /dev/sde
    ```
- Paso 2

  Comprobamos que se haya cambiado al disco de repuesto:
   ```bash
    $ mdadm --detail /dev/md126
    $ mdadm --detail /dev/md127
    ```

## Prueba fichero modificado

- Paso 1

  Creamos un archivo random:
  ```bash
    $ dd if=/dev/urandom of=prueba.txt bs=500MB count=1
    ```
- Paso 2

  Metemos el hash md5 del fichero en otro fichero:
  ```bash
    $ md5sum prueba.txt > checksum.txt
    ```
- Paso 3

  Modificamos el archivo de prueba:
  ```bash
    $ echo "Modificado" >> prueba.txt
    ```
- Paso 4

  Comparamos ambos archivos para comprobar que el hash ha cambiado:
   ```bash
    $ md5sum -c checksum.txt
    ```

## LVM

- Paso 1

  Convertimos el RAID 10 en un volúmen físico:
  ```bash
    $ pvcreate /dev/md125
    ```
- Paso 2

  Creamos un grupo de volúmenes y añadimos el RAID 10:
  ```bash
    $ vgcreate SeguridadAD /dev/md125
    ```
- Paso 3

  Creamos dos volúmenes lógicos:
  ```bash
    $ lvcreate -l 70%FREE -n P1 SeguridadAD
    $ lvcreate -l 30%FREE -n P2 SeguridadAD
    ```
- Paso 4

  Formateamos los volúmenes lógicos para que tengan el sistema de archivos Ext4:
   ```bash
    $ mkfs.ext4 /dev/SeguridadAD/P1
    $ mkfs.ext4 /dev/SeguridadAD/P2
    ```
- Paso 5

  Creamos puntos de montaje:
  ```bash
    $ mkdir -p /datos001
    $ mkdir -p /datos002
    ```
- Paso 6

  Montamos los volúmenes lógicos en los directorios que acabamos de crear:
  ```bash
    $ mount /dev/SeguridadAD/P1 /datos001
    $ mount /dev/SeguridadAD/P2 /datos002
    ```
