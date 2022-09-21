# **Contenedores**

## **Comandos y usos de los contenedores**

Ya hemos visto que para listar los diferentes contenedores debemos ejecutar el comando:
```bash
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS                    PORTS     NAMES
04eee8d6495b   nginx     "/docker-entrypoint.…"   3 days ago    Exited (137) 3 days ago             stoic_mclaren
09ad58c932c3   nginx     "/docker-entrypoint.…"   2 weeks ago   Exited (0) 2 weeks ago              example-web
```
Eliminamos los contenedores:
```bash
$ docker rm example-web
example-web
$ docker rm stoic_mclaren
stoic_mclaren
```
Creamos un contenedor nuevo.
```bash
$ docker run -d -p 8081:80 --name nginx_v1 nginx
```
Lo paramos:
```bash
$ docker stop nginx_v1
```
Para volverlo a activar:
```bash
$ docker start nginx_v1
```
El comando `start` también sirve para reiniciar el contenedor sin tener que pararlo previamente.

Si queremos acceder a la terminal del contenedor:
```bash
$ exec -ti nginx_v1 bash
root@4cd58e393385:/#
```
Para salir del contenedor:
```bash
root@4cd58e393385:/# exit
exit
```
Podemos crear un usuario dentro del contenedor:
```bash
root@4cd58e393385:/# adduser paco
Adding user `paco' ...
Adding new group `paco' (1000) ...
Adding new user `paco' (1000) with group `paco' ...
Creating home directory `/home/paco' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password:
passwd: password updated successfully
Changing the user information for paco
Enter the new value, or press ENTER for the default
        Full Name []:          
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
root@4cd58e393385:/# exit
exit
```
Ahora podemos iniciar el contenedor con ése usuario:
```bash
$ docker exec -u paco -ti nginx_v1 bash
paco@4cd58e393385:/$ 
```
Ésto puede ser interesante para permitir permiso de jecución a ciertas aplicaciones dentro del contenedor, por ejemplo.

Si queremos eliminar todos los contenedores:
```bash
$ docker rm -f $(docker ps -aq)
```

Podemos escribir en la cmd del container del siguiente modo:
```bash
$ docker run -dti --name [NOMBRE_CONT] [NOMBRE_IMG] echo [CONTENIDO]
```
Para comprobarlo:
```bash
$ docker logs [NOMBRE_CONT]
```

Si queremos un contenedor que se autodestruya al salir de la terminal, debemos ejecutar el comando:
```bash
$ docker run --rm -ti --name centos centos bash
```
## **Trabajar con variables de entorno**
Construimos la siguiente imagen:
```dockerfile
# Definimos SO
FROM centos
# Creamos variable de entorno
ENV prueba 1234
# Añadimos un usuario
RUN useradd paco
```
Montamos la imagen:
```bash
$ docker build -t env:v1 .
```
Levantamos un contenedor:
```bash
$ docker run -dti --name env-v1 env:v1
```
y accedemos desde el usuario paco:
```bash
$  docker exec -u paco -ti env-v1 bash
```
Ahora podemos pedirle que nos muestre el valor de la variable de entorno prueba:
```bash
[paco@1b6a23a67d19 /]$ echo $prueba
1234
```
Salimos y eliminamos el contenedor:
```bash
[paco@1b6a23a67d19 /]$ exit
exit
$ docker rm -f env-v1
```
Podemos insertar el valor de una variable de entorno por la línea de comandos:
```bash
$ docker run -dti -e prueba1=4321 --name env-v1 env:v1
```
y al acceder a él podremos ver que se ha guardado dentro del contenedor:
```bash
$ docker exec -u paco -ti env-v1 bash
[paco@7cf3b247045e /]$ echo prueba1
prueba1
[paco@7cf3b247045e /]$ echo $prueba1
4321
[paco@7cf3b247045e /]$
```
## **Gestión de recursos**
Podemos ver el consumo de recursos de un contenedor:
```bash
$ docker stats my-mongo
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT    MEM %     NET I/O     BLOCK I/O    PIDS
5c0b5bd35a20   my-mongo   0.39%     60.9MiB / 1.939GiB   3.07%     796B / 0B   0B / 229kB   33  
```
Podemos asignarle un puerto específico a un contenedor:
```bash
$ docker run -d --name my-mongo -p 8083:27017 mongo
```
También podemos limitar la memoria y el uso de cpus del constenedor:
```bash
$ docker run -d --name my-mongo-2 -m "500mb" mongo --cpuset-cpus 0-1 -p 8084:27017 mongo
```

## **Convertir un contenedor en una imagen**
Creamos un contenedor:
```bash
$ docker run -d -p 8081:80 --name nginx_v1 nginx
```

Lo convertimos a imagen:
```bash
$ docker commit nginx_v1 nginx-custom
```

y podemos usarla para lanzar otro contenedor.

## **Document root**
Es donde Docker ubica los elementos del container. Resulta útil cuando se quiere recuperar información de volúmenes creados.
Para saber donde se ubica, ejecutamos el comando:
```bash
$ docker info | findstr -i root
Docker Root Dir: /var/lib/docker
```
Dentro de la ubicación se podrán encontrar diferentes carpetas, entre las cuales habrá una de volúmenes con toda la información acerca de ellos.