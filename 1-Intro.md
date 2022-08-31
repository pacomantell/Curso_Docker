# Introducción

Docker es un sistema de contenedores que recrea el mismo entorno de ejecución de software en diferentes máquinas, lo que elimina los problemas de ejecución asociados al entorno.

Ésto ofrece ventajas sobre las máquinas virtuales, ya que los contenedores son mucho más ligeros que éstas, de modo que se puede llegar a tener una gran cantidad de contenedores sin afectar al rendimiento de la máquina.

## Funcionamiento básico de Docker.

Supongamos que tenemos tres bloques: el Cliente, el Docker Host y el Registro.

- El cliente es el que se encarga de generar imágenes, editarlas y ejecutarlas.

- El registry es el encargado de registrar ésas imágenes.

- El Docker Host se encarga de asociar una imágen a un contenedor, aunque también posee un registro interno. Todos éstos procesos son gestionados por el Docker daemon.

![https://www.josedomingo.org/pledin/assets/wp-content/uploads/2016/02/docker2.png](https://www.josedomingo.org/pledin/assets/wp-content/uploads/2016/02/docker2.png)

## Docker CLI

Veamos los comandos más básicos de Docker.

### Docker version.
Si queremos conocer la versión de Docker que estamos empleando, basta con ejecutar el comando:
```bash
docker --version
```
### Docker info.
Nos muestra todos los posibles comandos que pueden ejecutarse en Docker.

```bash
docker --info
```
### Docker search.
Mediante éste comando podemos realizar una búsqueda de las imágenes generadas:
```bash
docker search busybox
```
aunque resulta más útil buscarlas en el dockerhub.

### Creando nuestro primer contenedor

Abrimos una terminal y ejecutamos el comando
```bash
docker run ubuntu echo 'Hello World'
```

Lo que acabamos de hacer es crear un contenedor a partir de la imagen de Ubuntu que nos muestre por pantalla la palabra Hello World.

Si queremos ver los contenedores activos ejecutamos el comando:
```bash
docker ps
```
y para ver todos los contenedores, estén activos o no:

```bash
docker ps -a
```

## Docker images.
Éste comando busca las imágenes existentes:
```bash
docker images
```

## Ejecutar en segundo plano.
Añadiendo el comando `--detach` es posible ejecutar el contenedor en segundo plano.

## Personalizar contenedor
Si queremos definir un nombre a un contenedor ejecutamos:
```bash
docker run --name [NOMBRE] --publish 
```
Se pueden renombrar contenedores del siguiente modo:
```bash
docker rename [OLD NAME] [NEW NAME]
```
## Ejecutar un contenedor interactivo.
Es posible crear un contendor interactivo, en el que podamos entrar en él y usar una terminal. Para ello, ejecutamos el comando:
```bash
docker run -it [NOMBRE_IMAGEN]
```
Por ejemplo:
```bash
docker run -it ubuntu
root@6be9dddeeea8:/# echo "Hello World!"
Hello World!
root@6be9dddeeea8:/# exit
exit
PS C:\Users\Raquel\Documents\Curso_Docker> 
```

Si queremos mantener el contenedor abierto sin misión:
```bash
docker run -dt [NOMBRE]
```

Para ejecutar un comando dentro del contenedor:
```bash
docker exec [NOMBRE CONTENEDOR] [COMANDO]
```

## Probando diferentes comandos.
Listar Imágenes:
```bash
docker images
```
Listar los diferentes contenedores:
```bash
docker container ls
```
o bien:
```bash
docker ps -all
```
Si queremos copiar contenido dentro de un contenedor:
```bash
docker cp [RUTA_ORIGEN] [CONTENEDOR][RUTA_DESTINO]
```
Para hacerlo al revés es igual.

Si queremos parar el contenedor:
```bash
docker stop [NOMBRE_CONTENEDOR]
```
Y para activarlo:
```bash
docker start [NOMBRE_CONTENEDOR]
```

Si queremos eliminar un contenedor (debe estar desactivado):
```bash
docker rm [NOMBRE CONTENEDOR]
```