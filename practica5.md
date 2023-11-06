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
  ```bash
    $ sudo ln -s /usr/share/easy-rsa/* ~/easy-rsa/
    ```

- Paso 4

  Restringimos el acceso para que solo el propietario pueda acceder a él:
  ```bash
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
  ```bash
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

## Generar certificado

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

## Crear CSR - Peticion

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
    $ sudo openssl genrsa -out barbara-server.key
    ```

- Paso 3

  Creamos la petición:
  ```bash
    $ openssl req -new -key barbara-server.key -out barbara-server.req
    ```

## Firmado por CA - Firmar petición

- Paso 1

  Nos movemos a la carpeta del CA e importamos la CSR:
   ```bash
    $ cd ~/easy-rsa
    $ ./easyrsa import-req barbara-server.req barbara-server
    ```

- Paso 2

  Firmamos la CSR:
   ```bash
    $ ./easyrsa sign-req server barbara-server
    ```

## Configurar Apache

- Paso 1

  Modificamos el archivo /etc/apache2/sites-available/default-ssl.conf:
  ```bash
    $   sudo nano /etc/apache2/sites-available/default-ssl.conf
    SSLCertificateFile      /etc/ssl/certs/barbara-server.crt
    SSLCertificateKeyFile   /etc/ssl/private/barbara-server.key
    ```

  










## Comprobación
