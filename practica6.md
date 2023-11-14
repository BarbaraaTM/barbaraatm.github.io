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

    Ver * [Práctica 5 - Transmisiones web seguras (TLS)](./practica5.md)



 
    
