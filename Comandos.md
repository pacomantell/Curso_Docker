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