# Trabajo Final SGD

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
