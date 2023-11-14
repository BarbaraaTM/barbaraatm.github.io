# Hardening Apache

## Introducción

En el extenso terreno de la ciberseguridad, resguardar la integridad y confidencialidad de la información en línea se convierte en una necesidad crucial, y ello implica la protección efectiva de los servidores web. Entre la diversidad de servidores web en funcionamiento, Apache destaca como uno de los más populares y extensamente utilizados. No obstante, esta popularidad también atrae una atención más aguda por parte de agentes malintencionados, subrayando la imperiosa necesidad de reforzar la seguridad de Apache mediante rigurosas prácticas de hardening.

## Ocultar la versión de Apache y la información del sistema operativo

- Paso 1

  Modificamos el archivo /etc/httpd/conf/httpd.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         ServerTokens Prod
         ServerSignature Off
  ```

- Paso 2

  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

## Deshabilitar el listado de directorios en Apache

- Paso 1
 
  Para demostrar la utilidad de este hardening necesitamos crear el siguiente directorio y ficheros:
     ```bash
    $ mkdir -p /var/www/html/test
    $ cd /var/www/html/test
    $ sudo touch app.py main.py
    ```

- Paso 2
 
  Modificamos el archivo /etc/httpd/conf/httpd.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         <Directory "/opt/apache/htdocs">
             Options -Indexes
         </Directory>
  ```

- Paso 3
 
  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

## Utilice el cifrado HTTPS en Apache

  Ver [Práctica 5 - Transmisiones web seguras (TLS)](./practica5.md)

## Habilitar la seguridad de transporte estricta HTTP (HSTS) para Apache

- Paso 1

  Habilitamos el moódulo de cabeceras:
   ```bash
    $ sudo a2enmod headers
  ```

- Paso 2

  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

- Paso 3

  Modificamos el archivo /etc/apache2/sites-available/default-ssl.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano  /etc/apache2/sites-available/default-ssl.conf
         <VirtualHost *:443>
            # .....
            # ....
            Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
         </VirtualHost>
  ```

- Paso 4

  Reiniciamos para que se apliquen los cambios:
   ```bash
    $ systemctl restart apache2
  ```

## Habilitar HTTP/2 en Apache:

- Paso 1

  Habilitamos el moódulo de HTTP2:
   ```bash
    $ sudo a2enmod http2
  ```

- Paso 2

  Modificamos el archivo /etc/apache2/sites-available/default-ssl.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano  /etc/apache2/sites-available/default-ssl.conf
         <VirtualHost *:443>
            Protocols h2 http/1.1
         </VirtualHost>
  ```

- Paso 3

  Reiniciamos para que se apliquen los cambios:
     ```bash
        $ systemctl restart apache2
     ```

- Paso 4
 
  Comprobamos con el siguiente comando:
     ```bash
        $ curl -I --http2 -s https://barbara-server | grep HTTP
     ```

## Deshabilitar la directiva ServerSignature en Apache

- Paso 1

  Modificamos el archivo /etc/httpd/conf/httpd.conf y comentamos la siguiente linea:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         # ServerSignature Off
  ```

- Paso 2

  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

## Establecer la directiva 'ServerTokens' en 'Prod'

- Paso 1

  Modificamos el archivo /etc/httpd/conf/httpd.conf y comentamos la siguiente linea:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         ServerTokens Off
  ```

- Paso 2

  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

## Deshabilitar módulos innecesarios

- Paso 1

  Visualizamos todos los módulos que tenemos habilitados:
   ```bash
    $ apache2ctl -M
  ```

- Paso 2

  Para deshabilitar un módulo debemos utilizar el siguiente comando:
   ```bash
    $ sudo a2dismod módulo
  ```

- Paso 3

  Reiniciamos para que se apliquen los cambios:
     ```bash
        $ systemctl restart apache2
     ```

## Limitar el tamaño de carga de archivos en Apache

- Paso 1
 
  Modificamos el archivo /etc/httpd/conf/httpd.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         <Directory "/opt/apache/htdocs">
             LimitRequestBody 4194304
         </Directory>
  ```

- Paso 2
 
  Reiniciamos el servicio de Apache:
   ```bash
    $ systemctl restart apache2
  ```

## Ejecutar Apache como un usuario y grupo separados

- Paso 1

  Primero creamos un nuevo usuario y un nuevo grupo:
   ```bash
    $ sudo groupadd apachegroup
    $ sudo useradd -g apachegroup apacheuser
  ```

- Paso 2

  Modificamos el archivo /etc/httpd/conf/httpd.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         User apacheuser
         Group apachegroup
  ```

- Paso 3

  Cambiamos los permisos del archivo:
  ```bash
    $ sudo chown -R apacheuser:apachegroup /var/www/html
  ```

- Paso 4

  Reiniciamos para que se apliquen los cambios:
     ```bash
        $ systemctl restart apache2
     ```

## Proteger los ataques DDOS y el Hardening

- Paso 1

  Modificamos el archivo /etc/httpd/conf/httpd.conf y añadimos las siguientes lineas:
   ```bash
    $ sudo nano /etc/httpd/conf/httpd.conf
         Tiemout 400
         MaxKeepAliveRequests 100
         KeepAliveTimeout 5
         LimitRequestFields 100
         LimitRequestFieldSize 100
  ```

- Paso 2

  Reiniciamos para que se apliquen los cambios:
     ```bash
        $ systemctl restart apache2
     ```


  
