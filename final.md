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

### FW01 - El acceso a la red interna está controlado por el firewall

- Paso 1

  Vamos crear una regla que rechace todo el tráfico forward:
   ```bash
     $ sudo iptables –P FORWARD DROP
   ```

- Paso 2

  Vamos a aceptar el tráfico que salga de la DMZ para que pueda salir hacia afuera:
   ```bash
     $ sudo iptables -A FORWARD -i enp0s9 -o enp0s3 -j ACCEPT
   ```

- Paso 3

  De lo que venga de fuera, solo podrá acceder a la DMZ y a la red interna los paquetes con estado ESTABLISHED o RELATED:
   ```bash
     $ sudo iptables –A FORWARD –i enp0s3 –o enp0s9 –m state –state ESTABLISHED.RELATED -j ACCEPT
     $ sudo iptables –A FORWARD –i enp0s3 –o enp0s8 –m state –state ESTABLISHED.RELATED -j ACCEPT
   ```

- Paso 4

  Para que se apliquen las reglas deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```

### FW02 - Se permite la navegación desde la red interna 

- Paso 1

  Aceptamos el tráfico que salga de la red interna para que pueda salir hacia afuera con la siguiente regla:
  ```bash
     $ sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
   ```

- Paso 2

  Para que se aplique la regla deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```

### FW03 - Se permite el acceso al servidor web desde exterior

- Paso 1

  Aceptamos los paquetes que vengan de fuera por los puertos 80 y 443:
   ```bash
     $ sudo iptables -A FORWARD -i enp0s3 -o enp0s9 –p tcp –m multiport –dports 80,443 -j ACCEPT
   ```

- Paso 2

  Redirigimos todo el tráfico TCP que llega al equipo en los puertos 80 y 443 de la interfaz enp0s3 hacia la dirección IP 172.16.81.10.
  ```bash
     $ sudo iptables -t nat -A PREROUTING -i enp0s3 -p tcp -m multiport --dports 80,443 -j DNAT --to-destination 172.16.81.10 
   ```

- Paso 3

  Para que se apliquen las reglas deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```

### FW04 - El acceso a la red interna no está permitido desde la DMZ 

- Paso 1

  Aceptamos el tráfico que salga de la red interna hacia la DMZ con la siguiente regla:
   ```bash
     $ sudo iptables -A FORWARD -i enp0s8 -o enp0s9 -j ACCEPT 
   ```

- Paso 2

  De lo que venga de la DMZ, solo podrá acceder a la red interna los paquetes con estado ESTABLISHED o RELATED:
   ```bash
     $ sudo iptables –A FORWARD –i enp0s9 –o enp0s8 –m state –state ESTABLISHED.RELATED -j ACCEPT
   ```

- Paso 3

  Rechazamos el resto de paquetes que puedan venir desde la DMZ:
   ```bash
     $ sudo iptables -A FORWARD -i enp0s9 -o enp0s8 -j DROP 
   ```

- Paso 4

  Para que se apliquen las reglas deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```

