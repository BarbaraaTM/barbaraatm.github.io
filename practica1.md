# Logs centralizados

## Introducción
 El trabajo del administrador de sistemas pude verse simplificado controlando los logs (registro de procesos del sistema)
 Para ello utilizaremos la herramiento rsyslog.
## Desarrollo
Para facilitar el trabajo del administrador, vamos a hacer que los logs del cliente, también se guarden en el servidor.

**Configuración servidor**
- Paso 1

    Instalamos el paquete rsyslog:
    ```bash
    $ apt-get install rsyslog
    ```
- Paso 2

    Modificamos el fichero /etc/rsyslog.conf :
    ```bash
    $ nano /etc/rsyslog.conf
    ```
    Descomentamos las siguientes líneas:
    ```bash
    $ModLoad imudp
    $UDPServerRun 514
    ```
- Paso 3

    Reiniciamos el servicio:
    ```bash
    $ systemctl restart rsyslog
    ```
- Paso 4
  
    Configuramos el cortafuegos para que permita la comunicacióncon por el puerto 514:
    ```bash
    $ ufw allow 514/tcp
    $ ufw allow 514/udp
    ```
    Y reiniciamos el cortafuegos:
    ```bash
    $ ufw reload
    ```

**Configuración del cliente**
- Paso 1

    Modificamos el fichero /etc/rsyslog.conf :
    ```bash
    $ nano /etc/rsyslog.conf
    ```
    Añadimos la siguiente línea al final del archivo:
    ```bash
    *.* IP_SERVIDOR:514
    ```
- Paso 2

    Reiniciamos el servicio:
    ```bash
    $ systemctl restart rsyslog
    ```

**Comprobación**

Desde el cliente escribimos el siguiente comando:
 ```bash
    $ logger "prueba"
```

Ahora nos vamos al servidor y escribimos:
```bash
    $ tail /var/log/syslog
```
Y deberíamos ver un mensaje parecido a este:
