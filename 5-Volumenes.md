# Volúmenes

## **Tipos de volúmenes.**
Los volúmenes en Docker nos permiten persistir datos dentro de los contenedores, lo cual resulta muy útil a la hora de trabajar con bases de datos (BBDD).
En Docker existen tres tipos de volúmenes distintos:
- **host**
    - Se definen de manera local del siguiente modo:
    ```bash
    $ docker run -d -v <localizacion-local>:<localizacion-docker>
    ```
    Son los más cómodos de emplear dada su portabilidad.
- **anonymus**
    - Éste tipo de volúmenes son gestionados de manera interna por docker, de modo que tan sólo se debe especificar qué parte dentro del contenedor ha de persistir.
    ```bash
    $ docker run -d -v <localizacion-docker>
    ```
- **named volumes**
    - Son muy similares a los anónimos, pero a éstos podemos asignarles un nombre.
    Para crearlos, basta con ejecutar el comando:
    ```bash
    $ docker volume create <volume-name>
    ```
    Una vez creados, les podemos asignar una parte del contenedor que queremos que persista:
    ```bash
    $ docker run -d -v <volume-name>:<localizacion-docker>
    ```

Cabe destacar que tanto los volúmenes anónimos como los nombrados se alojan dentro de la estructura interna de docker.

Para ver las doferencias entre cada tipo de volúmenes, veámos unos ejemplos.

## **Trabajando con tipos de volúmenes en contenedores MySQL**

### **Volúmen host**

Creamos una imagen de MySQL:
```bash
$ docker pull mysql:5.7
```
Creamos un contenedor, tendiendo en cuenta que para la imagen de MySQL es necesario especificar variables de entorno, creando un volumen de tipo host, ya que será el más útil dada su portabilidad.
```bash
$ docker run -d -p 80:3306 --name my-db -e "MYSQL_ROOT_PASSWORD=12345678" -v C:\Users\Raquel\Documents\Curso_Docker\mysql:/var/lib/mysql mysql:5.7
```
e iniciamos sesión:
```bash
$ docker logs -f my-db
```
Podemos entrar dentro del contenedor:
```bash
$ docker exec -ti my-db bash
bash-4.2#
```
e iniciar sesión en MySQL:
```bash
bash-4.2# mysql -u root -p12345678
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
Una vez iniciada sesión, podemos ver las bases de datos disponibles:
```bash
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```
Ahora creamos una base de datos nueva:
```bash
mysql> CREATE DATABASE test1;
Query OK, 1 row affected (0.02 sec)
```
Salimos:
```bash
mysql> exit
Bye
bash-4.2# exit
exit
```
Si ahora eliminamos el contenedor y volvemos a levantarlo asignando el mismo volumen:
```bash
$ docker rm -f my-db
my-db
$ docker run -d -p 80:3306 --name my-db -e "MYSQL_ROOT_PASSWORD=12345678" -v C:\Users\Raquel\Documents\Curso_Docker\mysql:/var/lib/mysql mysql:5.7
b15824f1f2e236a3a87f465c731d3a215b199d9ad1b9073185250c454bc809e6
```
podremos ver que, accediendo a MySQL dentro del contenedor, la base de datos creada anteriormente sigue estando, de modo que ha persistido en el volumen creado:
```bash
$ docker exec -ti my-db bash
bash-4.2# mysql -u root -p12345678
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
+--------------------+
5 rows in set (0.12 sec)
```
### **Volumen anonymus**
Creamos un contenedor con un volumen de tipo anónimo:
```bash
$ docker run -d --name db -p 80:3306 -e "MYSQL_ROOT_PASSWORD=12345678" -v /var/lib/mysql mysql:5.7
```

Si ahora eliminamos el contenedor, se perderán todos los datos del volumen, pero podemos recuperarlos accediendo al document root del volumen. Para ello, basta con inspeccionar el contenedor y obtener la dirección del volumen en el document root. De ése modo, si tumbamos el contenedor y volvemos a levantarlo especificando dicha ruta, podremos recuperar los datos almacenados en el volumen:
```bash
$ docker run -d --name db -p 80:3306 -e "MYSQL_ROOT_PASSWORD=12345678" -v /var/lib/docker/volumes/10a8f7e053ef1514cbe2f59ef25b5d601d0e35a06968ddbc53d5824556d63463/_data:/var/lib/mysql mysql:5.7
```

### **Volumen named**

Creamos en primer lugar un volumen:
```bash
$ docker volume create mysql-data
```
y se lo asignamos a un contenedor:
```bash
$ docker run -d --name db -p 80:3306 -e "MYSQL_ROOT_PASSWORD=12345678" -v mysql-data:/var/lib/mysql mysql:5.7
```
Entramos dentro del contenedor e iniciamos sesión en MySQL:
```bash
$ docker exec -ti db bash
bash-4.2# mysql -u root -p12345678
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
Creamos un par de bases de datos:
```bash
mysql> CREATE DATABASE test1;
Query OK, 1 row affected (0.00 sec)
mysql> CREATE DATABASE test2;
Query OK, 1 row affected (0.00 sec
```
Podemos comprobar que se hayan creado:
```bash
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
| test2              |
+--------------------+
6 rows in set (0.00 sec)
```
Ahora salimos y eliminamos el contenedor:
```bash
mysql> exit
Bye
bash-4.2# exit
exit
$ docker rm -f db
db
```

Si volvemos a levantarlo y entramos dentro, podrmeos ver cómo las dos bases de datos creadas han persisitido:
```bash
$ docker run -d --name db -p 80:3306 -e "MYSQL_ROOT_PASSWORD=12345678" -v mysql-data:/var/lib/mysql mysql:5.7
cc2c103e1fcabc29da93771641faa68c77ea02ed6fb29046cdec01927be94929
$ docker exec -ti db bash
bash-4.2# mysql -u root -p12345678
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.39 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
| test2              |
+--------------------+
6 rows in set (0.00 sec)
```
## **Compartiendo volúmenes**
Creamos el siguiente dockerfile:
```dockerfile
FROM centos
# Creamos una carpeta
RUN mkdir /start
# Copiamos un script a la carpeta
COPY start.sh /start
# Damos permiso al script
RUN chmod +x /start/start.sh
# Ejecutamos el script
CMD /start/start.sh
```
siendo el script el siguiente:

```bash
#!/bin/bash
while true; do
    echo "<p>$(date +%H:%M:%S)</p>" > /opt/index.html
    sleep 10
done
```
Montamos la imagen:
```bash
$ docker build -t vol .
```
Creamos un contenedor:
```bash
$ docker run -d --name generator -v C:\Users\Raquel\Documents\Curso_Docker\05-Almacenamiento\common:/opte vol
```
Si accedemos dentro de él al archivo index.html, podremos ver cómo se ha escrito la fecha, tal y como debía hacer el script:
```bash
$ docker exec -ti generator bash
[root@f13daf5b1864 /]# cd opt
[root@f13daf5b1864 opt]# ls -l
total 4
-rw-r--r-- 1 root root 16 Sep 21 12:41 index.html
[root@f13daf5b1864 opt]# cat index.html 
<p>12:41:37</p>
[root@f13daf5b1864 opt]#
```
De éste modo, podemos ver cómo un mismo volumen se ha compartido con un contenedor, pudiéndose extender a más contenedores.

## **Instrucciones docker para volúmenes**
Para crear un volumen:
```bash
$ docker volume create <nombre_vol>
```

Para listar los volúmenes:
```bash
$ docker volume ls
```

Para eliminar todos los volúmenes:
```bash
$ docker volume prune
```
Para eliminar un volumen:
```bash
$ docker volume rm [nombre]
```