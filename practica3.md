# Permisos / Reglas ACL

## Introducción
Vamos a realizar una situación ficticia de la estructura de directorios de un instituto.

![imágenes](./img/tree.png)

Existiran los siguientes grupos y usuarios:
|Usuarios|Grupos|
|-|-|
|t1|teachers|
|t2|teachers|
|s1|students, eso1|
|s2|students, eso2|
|s3|students|

## Desarrollo estructura

- Paso 1

    Creamos los usuarios:
    ```bash
    $ adduser *nombreUsuario*
    ```
- Paso 2

    Creamos los grupos:
    ```bash
    $ addgroup *nombreUsuario*
    ```
- Paso 3

   Añadimos los usuarios a sus respectivos grupos:
    ```bash
    $ adduser *nombreUsuario* *nombreGrupo*
    ```
- Paso 4

    Creamos los directorios:
    ```bash
    $ mkdir *nombreDirectorio*
    ```

## Implantación de permisos

- Paso 1

    A la carpeta *teachers*, vamos a darle permisos al grupo *teachers* de lectura, y al usuario *t1* de lectura y escritura.
    ```bash
    $ setfacl u:t1:rw,g:teachers:r
    ```
- Paso 2

    A la carpeta *students*, vamos a darle permisos al grupo *teachers* de lectura y escritura, y al grupo *students* de lectura.
     ```bash
    $ setfacl g:students:r,g:teachers:rw
    ```
- Paso 3

    A la carpeta *eso1*, vamos a darle permisos al grupo *teachers* de lectura y escritura, y al grupo *eso1* de lectura.
     ```bash
    $ setfacl g:eso1:r,g:teachers:rw
    ```
- Paso 4

    A la carpeta *eso2*, vamos a darle permisos al grupo *teachers* de lectura y escritura, y al grupo *eso2* de lectura.
     ```bash
    $ setfacl g:eso2:r,g:teachers:rw
    ```