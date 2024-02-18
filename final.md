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

- Paso 1

  Primero que nada actualizaremos los paquetes con las versiones más recientes:
  ```bash
   $ sudo apt-get update && sudo apt-get full-upgrade
  ```

- Paso 2

  Ahora instalamos el servicio isc-dhcp-server:
  ```bash
   $ sudo apt-get install isc-dhcp-server
  ```

- Paso 3

  Modificaremos el archivo /etc/default/isc-dhcp-server, y especificaremos la interfaz por la que atenderá a los clientes:
  ```bash
   $ sudo nano /etc/default/isc-dhcp-server
        ...
        INTERFACESv4="enp0s8"
        ...
  ```

- Paso 4

  Editamos el archivo /etc/dhcp/dhcpd.conf y configuraremos lo que el servidor ofrecerá a los clientes:
  ```bash
   $ sudo nano /etc/dhcp/dhcpd.conf
        ...
        subnet 172.16.82.0 netmask 255.255.255.0 {
         range 172.16.82.50 172.16.82.90;
         option routers 172.16.82.1;
         option domain-name-servers 8.8.8.8;
         default-lease-time 600;
         max-lease-time 7200;
        }
        ...
  ```

- Paso 5
 
    Vamos a reiniciar el servicio y consultar que su estado es active:
    ```bash
     $ sudo systemctl restart isc-dhcp-server
     $ sudo systemctl status isc-dhcp-server
    ```


## Firewall

Para poder facilitarnos el proceso de aplicar las reglas de iptables, instalaremos el siguiente paquete:
 ```bash
     $ sudo apt-get install iptables-persistent
 ```

### FW00 - El Firewall realiza NAT correctamente

- Paso 1

  Primero activamos el enrutamiento modificando el archivo /etc/sysctl.conf y descomentando la siguiente línea:
   ```bash
     $ sudo nano /etc/sysctl.conf
          ...
          net.ipv4.ip_forward=1
          ...
   ```

- Paso 2

  Guardamos los cambios para que se inicie cada vez que el servidor se encienda:
  ```bash
     $ sudo sysctl -p 
  ```

- Paso 3

  Agregamos la siguiente regla de iptables para que realice NAT:
   ```bash
     $ sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
   ```

- Paso 4

  Para que se aplique la regla deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```
