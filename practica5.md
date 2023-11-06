# Transmisiones web seguras (TLS)

## Introducción
Internet es un canal no seguro, cualquier persona podría obtener nuestros datos, para evitar que esto suceda, ciertas comunicaciones con información confidencial deben ser cifradas. Para ello, vamos a generar un certificado que esté autofirmado para un servidor web Apache.

## Instalación Apache
sudo apt install apache2
sudo apt update

## Crear CA

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
  ``bash
    $ sudo ln -s /usr/share/easy-rsa/* ~/easy-rsa/
    ```

- Paso 4

  Restringimos el acceso para que solo el propietario pueda acceder a él:
  ``bash
    $ sudo chmod 700 /home/barbara/easy-rsa
    ```
- Paso 5

  Inciamos el PKI dentro del directory easy-rsa:
   ```bash
    $ cd ~/easy-rsa
    $ ./easyrsa init-pki
    ```

- Paso 6

  Creamos y editamos el fichero vars con los datos de la organización:
  ``bash
    $ sudo nano ~/easy-rsa/vars
      ~/easy-rsa/vars
      set_var EASYRSA_REQ_COUNTRY    "España"
      set_var EASYRSA_REQ_PROVINCE   "Valencia"
      set_var EASYRSA_REQ_CITY       "Valencia"
      set_var EASYRSA_REQ_ORG        "ASIR2SGD"
      set_var EASYRSA_REQ_EMAIL      "bartormon@alu.edu.gva.es"
      set_var EASYRSA_REQ_OU         "Community"
      set_var EASYRSA_ALGO           "ec"
      set_var EASYRSA_DIGEST         "sha512"

    ```

    






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







Crear CA
Crear CSR - peticion
Firmado por CA - firmar peticion
Configurar Apache - default-ssl



## Generar certificado

## Autofirma

## Instalación

## Comprobación
