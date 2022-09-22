## Comandos Docker

## - Version.
```bash
$ docker --version
```
## - Crear un contenedor.
```bash
$ docker run [OPCIONES] NOMBRE_IMAGEN [COMANDO] [...]
```
## - Ejecutar en segundo plano.
```bash
$ docker run --detach [...]
```
## - Personalizar nombre del contenedor.
```bash
$ docker run --name [NOMBRE] --publish [...]
```
## - Cambiar nombre de contenedor.
```bash
$ docker rename [OLD NAME] [NEW NAME]
```
## - Ejecutar contenedor con shell interactiva.
```bash
$ docker run -it [NOMBRE_IMAGEN]
```
## - Mantener el contenedor abierto sin misión:
```bash
$ docker run -dt [NOMBRE]
```
## - Ejecutar un comando dentro del contenedor:
```bash
$ docker exec [NOMBRE CONTENEDOR] [COMANDO]
```
## - Acceder al contenedor con un usuario definido dentro de él:
```bash
$ docker exec -u <username> -ti nginx_v1 bash
```
## - Consumo de recursos de un contenedor:
```bash
$ docker stats [NOMBRE]
```
## - Eliminar todos los contenedores:
```bash
$ docker rm -f $(docker ps -aq)
```
## - Contenedor que se autodestruya al salir de la terminal:
```bash
$ docker run --rm -ti --name centos centos bash
```
## - Escribir en la cmd del container:
```bash
$ docker run -dti --name [NOMBRE_CONT] [NOMBRE_IMG] echo [CONTENIDO]
```
## - Convertir contenedor en imagen:
```bash
$ docker commit [NOMBRE_CONT] [NOMBRE_IMG]
```
## - Listar Imágenes:
```bash
$ docker images
```
## - Listar los diferentes contenedores.
```bash
$ docker container ls
```
o bien:
```bash
$ docker ps -all
```
para mostrar los activos:
```bash
$ docker ps
```
## - Copiar contenido dentro de un contenedor o desde él.
```bash
$ docker cp [RUTA_ORIGEN] [RUTA_DESTINO]
```
## - Parar el contenedor.
```bash
$ docker stop [NOMBRE_CONTENEDOR]
```
## - Activar contenedor.
```bash
$ docker start [NOMBRE_CONTENEDOR]
```

## - Eliminar un contenedor (debe estar desactivado):
```bash
$ docker rm [NOMBRE CONTENEDOR]
```

## - Imágenes huérfanas:
```bash
$ docker images --filter "dangling=true"
```

## - Cargar imágen sin ejecutar contenedor:
```bash
$ docker pull [NOMBRE_IMAGEN]
```

## Cargar versión específica de una imagen:
```bash
$ docker pull [NOMBRE_IMAGEN]:[VERSION]
```

## - Descargar todos los tags de una imagen:
```bash
$ docker pull -a [IMAGEN]
```

## - Descargar imagen desde otro registry:
```bash
$ docker run --rm [REGISTRY] echo "[IMAGEN]"
```

## - Filtrar por oficiales y numero de estrellas:
```bash
$ docker search --filter is-official=true --filter stars=3 [NOMBRE]
```

## - Ejecutar dockerfile:
```bash
$ docker build .
```

## - Nombrar la imagen al ejecutar un dockerfile:
```bash
$ docker build -t [NOMBRE_IMAGEN] .
```

## - Conocer el historial de distintas capas que posee una imagen:
```bash
$ docker history [IMAGEN]
```

## - Inspección de imágenes:
```bash
$ docker inspect [IMAGEN]
```

## - Eliminar una imágen.
```bash
$ docker rmi [IMAGEN]
```

## - Eliminar una imágen huérfana:
```bash
$ docker images -f dangling=true -q | xargs docker rmi
```

## - dirección IP de un contenedor:
```bash
$ docker container inspect [NOMBRE_CONT] | findstr "IPAddress"
```

## - Consultar el tipo de red de nuestras imágenes:
```bash
$ docker network ls
```

## - Establecer una comunicación entre contenedores (en la misma red):
```bash
$ docker exec centos2 bash -c "ping 172.17.0.2"
```

## - Eliminar todo el contenido de docker:
```bash
$ docker prune
```

## - Crear un volumen:
```bash
$ docker volume create <nombre_vol>
```

## - Listar los volúmenes:
```bash
$ docker volume ls
```

## - Eliminar todos los volúmenes:
```bash
$ docker volume prune
```
## - Eliminar un volumen:
```bash
$ docker volume rm [nombre]
```

## - Ejecutar un docker compose ejecutamos el siguiente comando en consola, dentro de la carpeta donde se encuentre el archivo:
```bash
$ docker-compose up -d
```

## - Si nuestro docker compose tiene otro nombre diferente:
```bash
$ docker-compose -f ./[nombre] up -d
```

## - Bajar el docker compose:
```bash
$ docker-compose down
```

## - Construir las imágenes que contienen los servicios del docker compose:
```bash
$ docker-compose build
```

## - Para reconstruirlas:
```bash
$ docker-compose build --no-cache
```