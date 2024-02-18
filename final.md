# Trabajo Final SGD

## Introducción

En nuestro futuro escenario, estaremos configurando un firewall en un servidor Ubuntu equipado con tres interfaces de red: `enp0s3` para NAT, que nos permitirá la conexión con Internet; `enp0s8`, destinada a nuestra red interna, donde planeamos albergar un punto de acceso (AP), un servidor LDAP y un cliente; y `enp0s9`, que expandiremos hacia una DMZ para situar un servidor web. Nuestro plan incluye la implementación de un proxy Squid que se encargará exclusivamente de filtrar el tráfico destinado a la red interna, con el objetivo de optimizar la seguridad y la gestión del tráfico mediante iptables, y al mismo tiempo mejorar la eficiencia de la navegación web, garantizando así un acceso controlado y seguro para nuestra infraestructura interna y los servicios expuestos en la DMZ.

![diagrama](img/diagrama~2.png)

## Preparando el servidor principal

Deberemos configurar la IP estática para cada una de las interfaces del servidor, y configuraremos el servicio DHCP para que le proporcione a los clientes de la red interna una IP de forma automática.

### Configuración del netplan

- Paso 1

  Modificamos el archivo /etc/netplan/00-installer-config.yaml y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/netplan/00-installer-config.yaml
         network:
           version: 2
           renderer: networkd
           ethernets:
             enp0s3:
               dhcp4: false
               addresses:
                - 192.168.82.225/24
               routes:
                - to: default
                  via: 192.168.82.100
               nameservers:
                 addresses: [8.8.8.8]
             enp0s8:
               dhcp4: false
               addresses:
                - 172.16.82.1/24
             enp0s9:
               dhcp4: false
               addresses:
                - 172.16.81.1/24
  ```

- Paso 2

  Aplicamos los cambios:
  ```bash
   $ sudo netplan apply
  ```

### Configurando el servicio DHCP


## Firewall

