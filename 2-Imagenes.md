# Trabajando con imágenes en Docker

## Imágenes huérfanas.
Cuando realizamos un cambio en un dockerfile y lanzamos la imagen, la versión anterior de dicha imagen pasará a estar en desuso (huérfana).
Por ejemplo, en el dockerfile de 01-quick_start, si añadimos el comando:
```dockerfile
RUN ls
```
y volvemos a lanzar la imagen, el listado será el siguiente:
```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
image-1      latest    0847745cfd5a   10 minutes ago   280MB
<none>       <none>    d1d720ced60a   12 minutes ago   280MB
nginx        latest    2b7d6430f78d   3 weeks ago      142MB
ubuntu       latest    df5de72bdb3b   6 weeks ago      77.8MB
```

Si queremos ver cuáles son las imágenes huerfanas, ejecutamos el comando:

```bash
$ docker images --filter "dangling=true"
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
<none>       <none>    d1d720ced60a   12 minutes ago   280MB
```
## Versiones.

Cargamos ahora la imagen hello-world sin ejecutar un contenedor:
```bash
$ docker pull hello-world
```

Si queremos un versión específica de una imagen, aplicamos el siguiente filtro ":version"
```bash
$ docker pull redis:[VERSION]
```

Para descargarnos todas las tags que posee una imagen:
```bash
$ docker pull -a [IMAGEN]
```
Para descargar una imagen de un registry diferente a dockerhub, podemos o bien cambiarlo en la configuración de docker, o bien podemos indicárselo:
```bash
$ docker run --rm [REGISTRY] echo "[IMAGEN]"
```
Para filtrar la busqueda de una imagen según sus estrellas:
```bash
$ docker search --filter stars=3 --no-trunc [NOMBRE]
```
y si sólo queremos versiones oficiales:
```bash
$ docker search --filter is-official=true --filter stars=3 [NOMBRE]
```
## Historial.
El comando `history` permite conocer el historial de distintas capas que posee una imagen.
```bash
$ docker history [IMAGEN]
```
## Inspección de imágenes.
Las imágenes pueden ser inspeccionadas, obteniéndose toda la información de las mismas:
```bash
$ docker inspect [IMAGEN]
```
## Eliminar una imágen.
Es posible eliminar imágenes que no están siendo utilizadas por ningún contenedor.
```bash
docker rmi [IMAGEN]
```
Para eliminar una imágen huérfana:
```bash
$ docker images -f dangling=true -q | xargs docker rmi
```

# Dockerfiles.
Un dockerfile es un fichero donde se imprime la receta a sehuir para construir una imagen. Para abrir dicho fichero, se debe ejecutar el comando:
```bash
$ docker build .
```
podemos elegir el nombre de la imagen:
```bash
$ docker build -t [NOMBRE_IMAGEN] .
```
Normalmente se suele llamar al fichero dockerfile, aunque puede tener otro nombre. De ser así, se debe especificar el mismo:
```bash
$ docker build -t [NOMBRE_IMAGEN] -f [NOMBRE_DOCKERFILE]
```
## Formato.
El formato de los dockerfiles suele ser el siguiente:
```dockerfile
# Esto es un comentario
# Se empieza por los FROM, es decir, qué paquetes necesitamos
FROM PAQUETE
# Capas de nuestra imagen (comandos).
RUN COMANDO
RUN DOMANDO2
...
```
## Buenas prácticas creando el dockerfile:
- Los contenedores deben ser efímeros.
- Uso de ficheros [.dockerignore](.dockerignore), que nos perfmitirá ignorar ciertos ficheros de nuestra imagen que no sean necesarios.
- No instalar paquetes innecesarios.
- Minimizar el número de capas.
- Las capas deben ejecutarse en múltiples líneas.

## Instrucciones de Dockerfile.
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

# Buenas prácticas.

```dockerfile
FROM nginx

# Ésto no sería correcto
LABEL version=1
RUN echo "1" >> /usr/share/nginx/html/test.txt
RUN echo "2" >> /usr/share/nginx/html/test.txt
RUN echo "3" >> /usr/share/nginx/html/test.txt

# Lo correcto sería así
LABEL version=2
RUN \
    echo "1" >> /usr/share/nginx/html/test.txt && \
    echo "2" >> /usr/share/nginx/html/test.txt && \
    echo "3" >> /usr/share/nginx/html/test.txt
```
## Multistage building
Permite construir imágenes que trabajan sobre los outputs de otras, permitiendo recuperar sus resultados, lo que agiliza el peso en memoria.
Un ejemplo sería:
```dockerfile
# Creo mi primera imagen y le asigno un alias
FROM maven:3.5-alpine as builder
# Copio contenido a la carpeta /app
COPY simple-java-maven-app /app
# ejecuto un comando dentro de la carpeta
RUN cd /app && mvn package
# Vuelvo a montar otra imagen
FROM openjdk:8-alpine
# A partir del output de la imagen builder, copia el contenido a una carpeta
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /opt/app.jar
# Ejecuto java
CMD java -jar /opt/app.jar
```