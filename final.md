# Trabajo Final SGD
_Trabajo realizado por: Bárbara Torres, Adrian Racuci y Abderrahman Zrig_

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


## Proxy

Estaremos utilizando en esta ocasión, el proxy Squid, es un servidor web proxy-caché con licencia GPL cuyo objetivo es funcionar como proxy de la red y también como zona caché para almacenar páginas web, entre otros.
 ```bash
     $ sudo apt-get install squid
 ```

### PROXY01 - Está configurado de forma transparente

- Paso 1

   Modificaremos el fichero /etc/squid/squid.conf, y escribiremos lo siguiente:
   ```bash
     $ sudo nano /etc/squid/squid.conf
          ...
          http_port 3128 transparent
          ...
   ```

- Paso 2

  Estableceremos la siguiente regla para redirigir el tráfico entrante del puerto 80 y 443 hacia el puerto 3128 que es el puerto escucha del proxy:
  ```bash
     $ sudo iptables -t nat -A PREROUTING -p tcp -m multiport --dports 80,443 -j REDIRECT --to-port 3128 
  ```

- Paso 3

  Para que se aplique la regla deberemos escribir los siguientes comandos:
  ```bash
     $ sudo netfilter-persistent save
     $ sudo netfilter-persistent reload
  ```

***ATENCIÓN: El proxy transparente no funciona de forma simultanea con el siguiente paso, solo podemos configurar uno a la vez en cada proxy, por lo que debemos elegir cual preferimos para nuestra configuración. En nustro caso, nos quedaremos con el siguiente paso y añadiremos de forma manual la IP de nuestro proxy en la configuración de nuestro navegador cliente.***

### PROXY02 - La navegación a través del proxy se hace previa autenticación con los usuarios de LDAP.

- Paso 1

  Modificaremos el fichero /etc/squid/squid.conf, y escribiremos lo siguiente:
   ```bash
     $ sudo nano /etc/squid/squid.conf
          ...
          auth_param basic program /usr/lib/squid/basic_ldap_auth -b "ou=People,dc=aba,dc=com" -f "uid=%s -h 172.16.82.15"
          auth_param basic children 5 startup=5 idle=1

          auth_param basic credentialsttl 2 hours
          acl ldap_users proxy_auth REQUIRED

          http_access allow ldap_users
          http_access allow all
          ...
   ```

- Paso 2

  Reiniciaremos el servicio squid para aplicar los cambios:
   ```bash
     $ sudo systemctl restart squid
   ```

### PROXY03 - Impide la navegación a webs específicas 

- Paso 1

  Modificaremos el fichero /etc/squid/squid.conf, y escribiremos lo siguiente:
   ```bash
     $ sudo nano /etc/squid/squid.conf
          ...
          acl sitios dstdomain .(sitio web especifico)

          http_access deny sitios
          http_access
          ...
   ```

- Paso 2

  Reiniciaremos el servicio squid para aplicar los cambios:
   ```bash
     $ sudo systemctl restart squid
   ```

## Preparando el servidor web de la DMZ

Deberemos configurar la IP estática para el servidor web, e instalaremos el servicio apache.

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
                - 172.16.81.10/24
               routes:
                - to: default
                  via: 172.16.81.1
               nameservers:
                 addresses: [8.8.8.8]
  ```

- Paso 2

  Aplicamos los cambios:
  ```bash
   $ sudo netplan apply
  ```

### Instalamos Apache

- Paso 1
 
  Instalamos el paquete apache:
  ```bash
   $ sudo apt-get install apache2
  ```

## Servidor Web

### WEB01 - Es accesible desde la red externa 

Este apartado ya ha sido resuelto en el apartado [FW03 - Se permite el acceso al servidor web desde exterior](#fw03---se-permite-el-acceso-al-servidor-web-desde-exterior).

### WEB02 - Sirve la página index.html

- Paso 1

  Modificamos el archivo /var/www/html/index.html y creamos la página web que queramos que se vea al acceder a nuestro sitio web:
  ```bash
   $  sudo nano /var/www/html/index.html
      <html>
      <head>
        <title>Web ABA</title>
        <meta cahrset="utf-8">
      </head>
      <body>
        <h1>Ejercicio Final Seguridad</h1>
        <p>Trabajo realizado por: Adrian Racuci, Bárbara Torres y Abderrahman Zrig.</p>
      </body>
      </html>
  ```

- Paso 2

  Reiniciamos el servicio Apache para aplicar los cambios:
  ```bash
    $ sudo systemctl restart apache2
  ```

### WEB03 - Sirve la documentación elaborada con un procesador de textos en formato PDF 

- Paso 1

  Creamos la carpeta /var/www/html/pdf:
   ```bash
    $ sudo mkdir /var/www/html/pdf
  ```

- Paso 2

  Movemos el pdf con la documentación al directorio que hemos creado antes:
   ```bash
    $ sudo mv /(ubicacion del pdf) /var/www/html/pdf
  ```

- Paso 3

  En el archivo de configuración /etc/apache2/apache2.conf, editamos la siguiente línea para poder dar acceso a Apache a documentos con la extensión .pdf:
  ```bash
   $ sudo nano /etc/apache2/apache2.conf
     ...
     AddType application/pdf .pdf
     ...
  ```

- Paso 4

  Reiniciamos el servicio Apache para aplicar los cambios:
  ```bash
    $ sudo systemctl restart apache2
  ```

- Paso 5

  Para comprobarlo tenemos que ir a un cliente y en el navegador ponemos "http://172.16.81.10/pdf/TrabajoSGD.pdf", y nos descargará automáticamente el archivo pdf.

### WEB04-EX - Sirve la documentación elaborada en formato MarkDown como HTML 

- Paso 1

  Instalamos el paquete de MarkDown:
  ```bash
    $ sudo apt install markdown
  ```

- Paso 2

  Movemos el md con la documentación al directorio que hemos creado antes:
   ```bash
    $ sudo mv /(ubicacion del md) /var/www/html/
  ```

- Paso 3

  Ahora haremos uso de los servicios del paquete que hemos instalado antes para cambiar la extensión de md a html:
   ```bash
    $ cd /var/www/html/
    $ sudo markdown TrabajoSGD.md > TrabajoSGD.html
  ```

- Paso 4

  Para comprobarlo tenemos que ir a un cliente y en el navegador ponemos "http://172.16.81.10/TrabajoSGD.html".

### WEB05-EX - Se lleva a cabo una conexión segura mediante el protocolo HTTPS mediante un certificado firmado por una autoridad NO reconocida.

#### Crear CA

- Paso 1

  Instalamos el conjunto de secuencias de comandos easy-rsa:
  ```bash
    $ sudo apt install easy-rsa
  ```
  
- Paso 2

  Preparamos un directorio para la infraestructura de clave pública:
  ```bash
    $ sudo mkdir ~/easy-rsa
    ```
  
- Paso 3

  Creamos enlaces simbólicos que apunten a los archivos del paquete easy-rsa:
  ```bash
    $ sudo ln -s /usr/share/easy-rsa/* ~/easy-rsa/
    ```

- Paso 4

  Restringimos el acceso para que solo el propietario pueda acceder a él:
  ```bash
    $ sudo chmod 700 /home/dmz/easy-rsa
    ```
- Paso 5

  Inciamos el PKI dentro del directory easy-rsa:
   ```bash
    $ cd ~/easy-rsa
    $ ./easyrsa init-pki
    ```

- Paso 6

  Creamos y editamos el fichero vars con los datos de la organización:
  ```bash
    $ sudo nano ~/easy-rsa/vars
      ~/easy-rsa/vars
      set_var EASYRSA_REQ_COUNTRY    "España"
      set_var EASYRSA_REQ_PROVINCE   "Valencia"
      set_var EASYRSA_REQ_CITY       "Valencia"
      set_var EASYRSA_REQ_ORG        "ABA"
      set_var EASYRSA_REQ_EMAIL      "aba@sgd.com"
      set_var EASYRSA_REQ_OU         "Community"
      set_var EASYRSA_ALGO           "ec"
      set_var EASYRSA_DIGEST         "sha512"

    ```

#### Generar certificado

- Paso 1

  Creamos el certificado root público y el par de claves privadas para su entidad de certificación:
  ```bash
    $ ./easyrsa build-ca
    ```

- Paso 2
 
  Copiamos todo el contenido del archivo ~/easy-rsa/pki/ca.crt:
     ```bash
    $ cat ~/easy-rsa/pki/ca.crt
     -----BEGIN CERTIFICATE-----
      MIIDSDCCAjCgAwIBAgIUC8eXCt5zFkhK8iVa5hb8IUsrPLowDQYJKoZIhvcNAQEL
      BQAwFTETMBEGA1UEAwwKQ0EtYmFyYmFyYTAeFw0yMzExMDIxMjAyMTlaFw0zMzEw
      MzAxMjAyMTlaMBUxEzARBgNVBAMMCkNBLWJhcmJhcmEwggEiMA0GCSqGSIb3DQEB
      AQUAA4IBDwAwggEKAoIBAQCz8agL3slcVVGPYWCiNvYEGPzbB+s1RbQmy8Ri5lSn
      cSWXXRfyiHFZD3h6gS+5IKvqFrfyxf/pkCdFJbNTK+PZa4vQZ2D+LUnA6e61oal4
      H17JZbRhV0hcHZVuhTkUpah51WS/4QCBl5m5WeWkrmaDx7I58HNT+VJi57hv82rv
      mUo29aVP0bnN8UOQvObC1K1UvzM3uY9pnawcbRcgJC5+djSKL6E3dBed3W6gnVah
      JXmBQJjVsyQqLFVkIErd5WrKHx4LyEyyWvsSWp7adS5t5/agDPMQrbhxeKgwVpwl
      CJyWIEj8VNZfbYVwWTfiVQ0FZ1ImV8tkvUXinvMTSEBtAgMBAAGjgY8wgYwwHQYD
      VR0OBBYEFHwXGwmoPmhPVbEkn6Y3k8QVRVAXMFAGA1UdIwRJMEeAFHwXGwmoPmhP
      VbEkn6Y3k8QVRVAXoRmkFzAVMRMwEQYDVQQDDApDQS1iYXJiYXJhghQLx5cK3nMW
      SEryJVrmFvwhSys8ujAMBgNVHRMEBTADAQH/MAsGA1UdDwQEAwIBBjANBgkqhkiG
      9w0BAQsFAAOCAQEAVFbBl63AdzFUfsGZYEwyA01qSW/iWqq493c+EGXMbHl4vj5k
      eeFQCfQnt96SbS/AyJ4ZKOsmZxkd9XTBuOJS+i20E50E4Sf2d2O0e+L4lFbhDObE
      Ry4CZWF0gCmVXLL0DC9R9R2roc22g3wqP+yD4VwsOWqQq0NLOpmesWUAfdal1aiR
      gAFjMMAZU4vQV8wp2DjYAvvi1FWCfXF6Ed6C+rwWfOx66hdxxnxRTgk/q9/z1KBH
      AgH1gcE68ZpGYwbjSIcayN2uE+CrhRMpwmSYGXAcFh1WWYdpgZdMvsiJoWA+8uKk
      04kkrLzJsy7jsuRdawMzaD2+I1oF1PoI11vd1A==
    -----END CERTIFICATE-----
    ```

- Paso 3

  Pegamos el certificado en el fichero /tmp/ca.crt:
  ```bash
    $ nano /tmp/ca.crt
    ```

#### Crear CSR - Peticion

- Paso 1

  Ejecutamos los siguientes comandos para asegurar que tenemos instalado el servicio openssl en el sistema:
  ```bash
    $ sudo apt update
    $ sudo apt install openssl
    ```

- Paso 2

  Creamos un directorio y generamos dentro una clave privada:
  ```bash
    $ sudo mkdir ~/practice-csr
    $ sudo cd ~/practice-csr
    $ sudo openssl genrsa -out aba-server.key
    ```

- Paso 3

  Creamos la petición:
  ```bash
    $ openssl req -new -key aba-server.key -out aba-server.req
    ```

#### Firmado por CA - Firmar petición

- Paso 1

  Nos movemos a la carpeta del CA e importamos la CSR:
   ```bash
    $ cd ~/easy-rsa
    $ ./easyrsa import-req aba-server.req aba-server
    ```

- Paso 2

  Firmamos la CSR:
   ```bash
    $ ./easyrsa sign-req server aba-server
    ```
  
#### Configurar Apache

- Paso 1

  Modificamos el archivo /etc/apache2/sites-available/default-ssl.conf:
  ```bash
    $   sudo nano /etc/apache2/sites-available/default-ssl.conf
    SSLCertificateFile      /etc/ssl/certs/aba-server.crt
    SSLCertificateKeyFile   /etc/ssl/private/aba-server.key
    ```

- Paso 2

  Movemos el certificado y la clave privada al directorio que hemos indicado antes:
  ```bash
    $ cd ~/practice-csr
    $ sudo cp aba-server.crt /etc/ssl/certs/
    $ sudo cp aba-server.key /etc/ssl/private/
    ```

- Paso 3

  Habilitamos el sitio y el módulo de SSL:
   ```bash
    $ sudo a2ensite default-ssl.conf
    $ sudo a2enmod ssl
    ```
  
- Paso 4

  Reiniciamos Apache:
   ```bash
    $ systemctl restart apache2
    ```


## Preparando el servidor LDAP de la red interna

Deberemos configurar la IP estática para el servidor web, e instalaremos el servicio ldap.

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
                - 172.16.82.15/24
               routes:
                - to: default
                  via: 172.16.82.1
               nameservers:
                 addresses: [8.8.8.8]
  ```

- Paso 2

  Aplicamos los cambios:
  ```bash
   $ sudo netplan apply
  ```

### Instalamos LDAP server

- Paso 1
 
  Instalamos los siguientes paquete:
  ```bash
   $ sudo apt-get install 
  ```

### LDAP01 - La gestión de LDAP se lleva a cabo mediante GUI

- Paso 1

Instalamos los paquetes necesarios para poder acceder al servidor LDAP mediante un entorno gráfico.

```bash
  $ sudo apt-get install sldap ldap-utils apache2 php libapache2-mod-php ldap-account-manager
```

- Paso 2

  Accedemos a la página web para poder editar LDAP: http://172.16.82.15/lam.

- Paso 3

  Una vez estamos en la interfaz de entrada, le damos a "Edit server profiles" y entramos con la contraseña que viene por defecto.

- Paso 4

  En "Server settings", cambiaremos la configuración que sale pre-determinada a la que tenemos nosotros. En Server address: 172.16.82.15; y en Tree suffix: dc=aba,dc=com.

### LDAP02 - El servidor RADIUS es capaz de autenticarse con los usuarios de LDAP

- Paso 1

  Instalaremos el paquete RADIUS:
  ```bash
  $ sudo apt-get install freeradius
  ```

- Paso 2

  Configuraremos el archivo /etc/freeradius/3.0/mods-available/ldap:
  ```bash
  $ sudo nano  /etc/freeradius/3.0/mods-available/ldap
       ...
       ldap {
       server = "aba.com"
       identity = "admin"
       password = "admin"
       ...
      }
      ...
  ```

- Paso 3

  Reiniciamos el servicio para que se apliquen las modificaciones:
  ```bash
  sudo systemctl restart freeradius
  ```

### LDAP03 - El servidor PROXY es capaz de autenticarse con los usuarios del directorio

Este apartado ya ha sido resuelto en el apartado [PROXY02 - La navegación a través del proxy se hace previa autenticación con los usuarios de LDAP](#proxy02---la-navegación-a-través-del-proxy-se-hace-previa-autenticación-con-los-usuarios-de-ldap).

### LDAP04 - El directorio incluye usuario invitado

- Paso 1

  Creamos un usuario llamado invitado en un fichero ldif:

  ```bash
  $ sudo nano invitado.ldif
       dn: uid=invitado,ou=usuarios,dc=aba,dc=com
       objectClass: top
       objectClass: person
       objectClass: organizationalPerson
       objectClass: inetOrgPerson
       uid: invitado
       cn: Usuario Invitado
       sn: Invitado
       givenName: Usuario
       userPassword: passwd
  ```

- Paso 2

  Añadimos el usuario al directorio y reiniciamos ldap:

  ```bash
  $ ldapadd -x -D "cn=admin,dc=aba,dc=com" -W -f invitado.ldif
  $ sudo systemctl restart sldap
  ```

- Paso 3

  Nos movemos al proxy. Añadimos la regla en el squid para que podamos autenticarnos con invitado, en el archivo de /etc/squid/squid.conf:
  ```bash
  $ sudo nano /etc/squid/squid.conf
       ...
       auth_param basic program /usr/lib/squid3/basic_ldap_auth -v 3 -b "dc=aba,dc=com" -D "cn=admin,dc=aba,dc=com" -W /path/to/passwordfile -f "(&(objectClass=posixAccount)(!(uid=invitado)))"
       ...
  ```

- Paso 4

  Reiniciamos el servicio para aplicar los cambios:
  ```bash
  $ sudo service squid restart
  ```

## Preparando el AP de la red interna

Para poder cambiar la IP de nuestro AP, deberemos conectarlo a una máquina cliente que se encuentre en la misma subred que la del AP. Si el AP es nuevo, deberás buscar, según su modelo, cuál es la IP predeterminada de ese AP. En nuestro caso, la IP de fábrica es 192.168.1.1, por lo que nuestra máquina cliente deberá tener una IP de la subred 192.168.1.0/24. Si la IP por defecto no funciona, deberás reiniciar el AP.

### RAD-AP01 - El punto de acceso está configurado de forma correcta

- Paso 1

  Nos dirigimos a la máquina cliente y escribimos en el navegador: http://192.168.1.1.

- Paso 2

  Si la conectividad es la adecuada, pedirá que te autentiques. Busca, según el modelo del AP, su contraseña predeterminada.

- Paso 3

  Una vez autenticados, veremos una gran interfaz con varias pestañas. En este caso, nos quedamos con la página predeterminada, en la cual podemos cambiar la IP de nuestro AP. En nuestro caso vamos a indicarle la           siguiente IP: 172.16.82.20; la siguiente máscara de red: 255.255.255.0; y el gateway: 172.16.82.1. Aplicamos cambios.

- Paso 4

  Si la configuración ha ido bien, al cambiar la IP de la máquina cliente en modo automático (DHCP), ya deberían volver a encontrarse el AP y la máquina cliente en la misma subred. Para comprobarlo, nos volvemos a          dirigir al navegador y escribimos: http://172.16.82.20. Si nos aparece la interfaz de log in, todo ha ido correctamente, sino encuentra la IP deberás reiniciar el AP y volver a empezar los pasos.

### RAD-AP02 - Los clientes inalámbricos son capaces de autenticarse en el directorio mediante el protocolo RADIUS

- Paso 1

  Nos dirigimos a la pestaña que pone Wireless, y dentro de ese menú seleccionamos Radius, donde vamos a configurar el servicio. Cambiairemos los siguientes aspectos: MAC Radius Client: Enable; Radius Auth Server           Address: 172.16.82.15; Radius Auth Server Port: 1812; Password Format: Shared Key; Radius Auth shared Secret: admin.

- Paso 2

  Para que se puedan conectar debemos poner el SSID y configurar el tipo de seguridad (por ejemplo, WPA2-Enterprise) en las conexiones de red del cliente. Una vez puesto el SSID, pedirá el nombre de usuario y la            contraseña. Configurando eso se podrá conectar a nuestra red y así el usuario puede acceder a los recursos de nuestra red.

