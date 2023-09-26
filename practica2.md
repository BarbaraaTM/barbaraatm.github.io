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
    $ sudo nano /etc/security/pwquality.conf
    ```
- Paso 3

    Debemos elegir la política de contraseñas a configurar. Yo he escogido la siguiente:
    ```
    difok = 2 
    ...
    minlen = 6
    ...
    dcredit =- 1
    ...
    ucredit = -1
    ...
    ocredit = -1
    ...
    maxrepeat = 3
    ...
    retry = 3
    ```
    Con esta política habremos configurado que:
    - Tendrá 3 intentos antes de que el sistema devuelva un error.
    - Una longitud mínima de la contraseña 6 carácteres.
    - Si cambiamos la clave, como mínimo debe haber 2 diferencias.
    - Como mínimo, deberá tener 1 carácter en mayúscula, 1 dígito y 1 símbolo.
    - No podrá contener más de 3 carácteres iguales de forma consecutiva.
    
    Una vez que hayamos configurado el fichero de texto, lo guardamos y ya podremos comprobarlo.

## Comprobación

Para comprobar que nuestra contraseña es apta, podemos usar la herramienta *pwscore*:
*Contraseña incorrecta*
```bash
$ echo Hola | pwscore
```
Dará error ya que no cumple ni con la longitud mínima, ni con el número mínimo de dígitos y símbolos.

*Contraseña correcta*
```bash
$ echo H0laa%1 | pwscore
```
Esta contraseña esta cumpliendo la política de contraseñas que hemos configurado.