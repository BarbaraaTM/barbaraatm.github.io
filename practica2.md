# Política de contraseñas

## Introducción
La seguridad de nuestro sistema puede verse amenazada, una buena politica de contraseñas reforzará la seguridad. El objetivo de la práctica es entender y configurar con las directrices de seguridad. para ello, utilizaremos el módulo pam pw_password.

## Desarrollo

**Instalación**

Instalamos el paquete libpam-cracklib:
```
$ sudo apt install libpam-cracklib
```

**Configuración**

- Paso 1

    Creamos una copia de seguridad del archivo que vamos a modificar.
    ```bash
    $ sudo cp /etc/pam.d/common-password /root/
    ```
- Paso 2

    Entramos en el archivo de configuración para poder editarlo:
    ```bash
    $ sudo nano /etc/pam.d/common-password
    ```
- Paso 3

    Una vez que estamos dentro del editor, debemos localizar una línea similar a la siguiente:
    ```
    password    requisite                pam_cracklib.so retry=3 minlen=8 difok=3
    ```
- Paso 4

    Debemos elegir la política de contraseñas a configurar. Yo he escogido la siguiente:
    ```
    password    requisite                pam_cracklib.so retry=3 minlen=6 difok=2 ucredit=-1 dcredit=-1 ocredit=-1
    ```
    Con esta política habremos configurado que:
    - Tendrá 3 intentos antes de que el sistema devuelva un error.
    - Una longitud mínima de la contraseña 6 carácteres.
    - Si cambiamos la clave, como mínimo debe haber 2 diferencias.
    - Como mínimo, deberá tener 1 carácter en mayúscula, 1 dígito y 1 símbolo.
    
    Una vez que hayamos configurado el fichero de texto, lo guardamos y ya podremos comprobarlo.

## Comprobación