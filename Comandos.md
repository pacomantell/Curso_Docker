## Comandos Docker

## - Version.
```bash
docker --version
```
## - Crear un contenedor.
```bash
docker run [OPCIONES] NOMBRE_IMAGEN [COMANDO] [...]
```
## - Ejecutar en segundo plano.
```bash
docker run --detach [...]
```
## - Personalizar nombre del contenedor.
```bash
docker run --name [NOMBRE] --publish [...]
```
## - Cambiar nombre de contenedor.
```bash
docker rename [OLD NAME] [NEW NAME]
```
## - Ejecutar contenedor con shell interactiva.
```bash
docker run -it [NOMBRE_IMAGEN]
```
## - Mantener el contenedor abierto sin misión:
```bash
docker run -dt [NOMBRE]
```
## - Ejecutar un comando dentro del contenedor:
```bash
docker exec [NOMBRE CONTENEDOR] [COMANDO]
```
## - Listar Imágenes:
```bash
docker images
```
## - Listar los diferentes contenedores.
```bash
docker container ls
```
o bien:
```bash
docker ps -all
```
## - Copiar contenido dentro de un contenedor o desde él.
```bash
docker cp [RUTA_ORIGEN] [RUTA_DESTINO]
```
## - Parar el contenedor.
```bash
docker stop [NOMBRE_CONTENEDOR]
```
## - Activar contenedor.
```bash
docker start [NOMBRE_CONTENEDOR]
```

## - Eliminar un contenedor (debe estar desactivado):
```bash
docker rm [NOMBRE CONTENEDOR]
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
docker rmi [IMAGEN]
```

## - Eliminar una imágen huérfana:
```bash
$ docker images -f dangling=true -q | xargs docker rmi
```