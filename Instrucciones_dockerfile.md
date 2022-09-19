# Instrucciones de Dockerfile.
* ### FROM
    Determina la imagen a partir de la cual se construirá nuestra imagen customizada.
    Sintáxis:
    ```dockerfile
    FROM [IMAGEN]
    ```
* ### ARG
    Introduce algún argumento dentro de nuestro dockerfile que vaya a ser utilizado.
    ```dockerfile
    ARG [NOMBRE]=[VALOR]
    ```
    Por ejemplo, si queremos la última versión de una imagen, podemos indicarlo de la siguiente forma:
    ```dockerfile
    ARG CODE_VERSION=latest
    FROM [IMAGEN]:${CODE_VERSION}
    ```

* ### MAINTAINER
    Junto a label nos da información sobre la construcción de la imagen, como el autor, para qué esta hecha o quién se encarga del mantenimiento.
    ```dockerfile
    MAINTAINER <NOMBRE> <EMAIL>
    ```
* ### RUN
    Se encarga de ejecutar un comando en una nueva capa encima de la imagen y hacer un commit de los resultados.
    Normalmente se suele emplear como paso intermedio dentro de un dockerfile.
    ```dockerfile
    RUN [COMANDO]
    ```
    Ejemplo:
    Creamos un dockerfile:
    ```dockerfile
    # Definimos SO
    FROM ubuntu:16.04
    # Instalamos apache
    RUN apt-get update
    RUN apt-get install apache2 -y
    ```
    Montamos la imagen:
    ```bash
    $ docker build -t apache-custom .
    ```
* ### ENV
    Se encarga de configurar la variables de entorno locales del dockerfile.
    ```dockerfile
    ENV <key> <value>
    ```
    o bien
    ```dockerfile
    ENV <key>=<value>
    ```
* ### COPY y ADD
    Permiten copiar archivos desde una ruta especificada dentro de un contenedor.
    ```dockerfile
    ADD [FUENTE]...[DESTINO]
    ```
    o bien
    ```dockerfile
    ADD ["[FUENTE],...[DESTINO]"]
    ```
    **NOTA:** La diferencia entre `COPY` y `ADD` es que la última también permite copiar contenido desde una url. 
* ### CMD
    Proporciona principalmente valores por defecto a un contenedor en ejecución. Éstos valores pueden incluir un ejecutable u omitirlo, por ejemplo.
    Ésta instrucción tiene tres posibles sintáxis:
    1. Forma exec (la preferida):
        ```dockerfile
        CMD ["EJECUTABLE", "PARAM1", "PARAM2"]
        ```
    2. Como parámetros por defecto de `ENTRYPOINT`(se emplea cuando se especifica un `ENTRYPOINT`):
        ```dockerfile
        CMD ["PARAM1","PARAM2"]
        ```
    3. Forma shell:
        ```dockerfile
        CMD [COMANDO] [PARAM1] [PARAM2]
        ```
    Ejemplo:
    ```dockerfile
    # Instalamos ubuntu
    FROM ubuntu:16.04
    # Ejecutamos el comando date dentro de ubuntu
    CMD ["/bin/date"]
    ```
    Construimos la imagen:
    ```bash
    $ docker build -t test-cmd .
    $ docker run --name test-cmd test cmd
    ```
    De éste modo, con cada log en el contenedor, obtendremos la fecha y hora por pantalla.
* ### ENTRYPOINT
    Permite configurar un contenedor que se utilizará como ejecutable.
    ```dockerfile
    ENTRYPOINT ["exec", "param1", "param2"]
    ```
    **Nota:** Las instrucciones `CMD` y `ENTRYPOINT` pueden aparecer o bien ambas, o ninguna, o sólo una de ellas. De éste modo, si se especifican los dos, el `ENTRYPOINT` definirá el ejecutable que usará el `CMD`.

    Ejemplo:
    ```dockerfile
    # Instalamos ubuntu
    FROM ubuntu:16.04
    # Creamos un ENTRYPOINT que use el comando echo
    ENTRYPOINT ["/bin/echo"]
    # Imprimimos Hola por pantalla
    CMD ["Hola"]
    ```
    Construimos la imagen:
    ```bash
    $ docker build -t test-entrypoint .
    $ docker run --name test-entrypoint test entrypoint
    ```
* ### WORKDIR
    Permite modificar el directorio de trabajo de las instrucciones `RUN`, `CMD`, `ENTRYPOINT`, `COPY` y `ADD` que le sigan en el dockerfile.
    ```dockerfile
    WORKDIR /path/to/workdir
    ```
* ### EXPOSE
    Nos permite exponer un puerto de red para que sea utilizado por el contenedor.
    ```dockerfile
    EXPOSE <PUERTO> [<PUERTO>/<PROTOCOLO>...]
    ```
    Ejemplo:
    ```dockerfile
    # Definimos el SO
    FROM centos
    # Instalamos apache para centos (S.O.)
    RUN yum install httpd -y
    # Exponemos el puerto 80 del contenedor
    EXPOSE 80
    # Ejecutamos Apache en primer plano
    CMD apachectl -DFOREGROUND
    ```
* ### LABEL
    Se utiliza para añadir metadatos a nuestra imagen, como autor, version, descripción, etc.
    ```dockerfile
    LABEL <KEY>=<VALUE> <KEY>=<VALUE> <KEY>=<VALUE> ...
    ```
* ### USER
    Establece el usuario (`UID`) o grupo de ususarios (`GID`) que ejecutarán la imagen.
    ```dockerfile
    USER <NOMBRE_USUARIO>
    USER [:<USUARIOS>]
    ```
* ### VOLUME
    Permite crear un puntero en el host que apunta hacia una dirección del contenedor
    ```dockerfile
    VOLUME [UBICACION]
    ```