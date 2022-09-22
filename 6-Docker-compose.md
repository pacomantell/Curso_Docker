# **Docker Compose**

## **Conceptos básicos**
Docker compose es un tipo de archivo que, mediante un formato yaml, permite crear servicios que se podrían interconectar entre ellos.
Docker compose viene de serie instalado con Docker Desktop.

Un ejemplo de archivo docker sería el siguiente:
```yaml
version: "3.7"
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```
Los elementos principales son:

- version (Obligatorio): nos dice que tipo de parametros puede haber dentro.
- services (Obligatorio): donde se listan los diferentes servicios que se pueden tener. En el ejemplo anterior se tendrían los siguientes servicios:
    - redis
    - db
    - vote
    - result
    - worker
    - visualizer
- network (No obligatorio): nos permite definir redes.
- volumes (No obligatorio): nos permite definir volúmenes.

Para ejecutar un docker compose ejecutamos el siguiente comando en consola, dentro de la carpeta donde se encuentre el archivo:
```bash
$ docker-compose up -d
```

Si nuestro docker compose tiene otro nombre diferente:
```bash
$ docker-compose -f ./[nombre] up -d
```

Para bajar el docker compose:
```bash
$ docker-compose down
```

Para construir las imágenes que contienen los servicios del docker compose:
```bash
$ docker-compose build
```

y para reconstruirlas:
```bash
$ docker-compose build --no-cache
```

## **Trabajando con ficheros docker-compose**

Creamos el siguiente docker-compose:
```yaml
version: '3'
services:
 web:
  container_name: "nginx-v1"
  ports:
   - "8080:80"
  image: nginx
```

Subimos el archivo:
```bash
$ docker-compose up -d
Creating network "proyects_default" with the default driver
Creating nginx-v1 ... done
```

LO bajamos:
```bash
$ docker-compose down
Stopping nginx-v1 ... done
Removing nginx-v1 ... done
Removing network proyects_default
```

## **Variables de entorno**

Podemos crear variables de entorno:
```yaml
version: '3'
services:
 db:
  container_name: mysql
  ports:
   - 3306:3306
  image: mysql:5.7
  environment:
   - "MYSQL_ROOT_PASSWORD=12345678"
```

Podemos lanzar el archivo, entrar dentro del contenedor y trabajar como hemos hecho hasta ahora.
```bash
$ docker-compose up -d
Creating network "proyects_default" with the default driver
Creating mysql ... done

$ docker exec -ti mysql bash
bash-4.2# env
HOSTNAME=2f15a1e4a8b5
TERM=xterm
MYSQL_VERSION=5.7.39-1.el7
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
MYSQL_SHELL_VERSION=8.0.30-1.el7
SHLVL=1
HOME=/root
MYSQL_MAJOR=5.7
GOSU_VERSION=1.14
MYSQL_ROOT_PASSWORD=12345678
_=/usr/bin/env
```

Lo habitual es crear un archivo que incluya todas las variables de entorno, el cual se cargará en el archivo docker-compose. De ése modo, el docker compose anterior quedaría de la forma:
```yaml
version: '3'
services:
 db:
  container_name: mysql-v2
  ports:
   - 80:3306
  image: mysql:5.7
  env_file: common.env
```
siendo el archivo common.env:
```env
MYSQL_ROOT_PASSWORD=12345678
hola=hola12
```
de modo que ahora al lanzar el archivo y entrar al contenedor, podremos ver que se han cargado las variables de entorno definidas:
```bash
$  docker-compose up -d      
Creating network "env_file_default" with the default driver
Creating mysql-v2 ... done

$ docker exec -ti mysql-v2 bash
bash-4.2# env
HOSTNAME=65fe67383259
TERM=xterm
MYSQL_VERSION=5.7.39-1.el7
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
MYSQL_SHELL_VERSION=8.0.30-1.el7
SHLVL=1
HOME=/root
MYSQL_MAJOR=5.7
GOSU_VERSION=1.14
hola=hola12
MYSQL_ROOT_PASSWORD=12345678
_=/usr/bin/env
```

## **Volúmenes nombrados**

Creamos el siguiente docker compose:
```yaml
version: '3'
services:
  web:
    container_name: nginx-v2
    ports:
    - "8080:80"
    image: nginx
    volumes:
    - "vol2:/user/share/nginx/html"
volumes:
  vol2:
```

en el cual hemos definido un volumen llamado vol2. Lanzamos el archivo:
```bash
$ docker-compose up -d
Creating network "volum_default" with the default driver
Creating volume "volum_vol2" with default driver
Creating nginx-v2 ... done
```

y podremos comprobar que se ha creado el volumen:
```bash
$ docker volume ls
DRIVER    VOLUME NAME
local     5f399b9a63f5cb94e46df6a699391f0558e39a2967466a0d7d46dcc6a2be32b9
local     9330b7574367c3feb4ab0013b48ddc69457dd1d84e880e77f2ef1f7455f9c491
local     c473e1775dca920f868a0d9abf576afa2ca859be7c1cadf6a702638488f92ffd
local     volum_vol2
```

## **Volúmenes host**

En éste caso, el volumen se define del siguiente modo:
```yaml
version: '3'
services:
  web:
    container_name: nginx-v2
    ports:
    - "8080:80"
    image: nginx
    volumes:
      - "./data:/user/share/nginx/html"
volumes:
  vol2:
```
Al lanzar el archivo veremos cómo se crea un carpeta llamada data, corresponidente al volumen creado.

## **Networks**

Podemos crear nuestra red dentro del docker-compose del siguiente modo:
```yaml
version: '3'
services:
  web-1:
    container_name: nginx
    ports:
    - "8080:80"
    image: nginx
    networks:
      - net-web
networks:
  net-web:
```

Si lanzamos el archivo, podremos comprobar cómo se ha creado nuestra nueva red:
```bash
$ docker-compose up -d
Creating network "networks_net-web" with the default driver
Creating nginx ... done

$ docker network ls   
NETWORK ID     NAME               DRIVER    SCOPE
20c75936fe0d   bridge             bridge    local
a25d0070553e   host               host      local
f916e12debbe   networks_net-web   bridge    local
e956fde01cc0   none               null      local
```

Podemos crear otro contenedor más dentro del docker-compose:
```yaml
version: '3'
services:
  web-1:
    container_name: nginx-1
    ports:
    - "8080:80"
    image: nginx
    networks:
      - net-web
  web-2:
    container_name: nginx-2
    ports:
    - "8080:80"
    image: nginx
    networks:
      - net-web
networks:
  net-web:
```
Si lanzamos el archivo, ambos contenedores estarán conectados a la red que hemos creado. También se puede aplicar a volúmenes, por ejemplo.

Podemos especificar todas las características de red que queramos:
```yaml
version: '3'
services:
  test-1:
    container_name: test_1
    image: image
    networks:
      testing-net:
        ipv4_address: 172.28.1.1
  
  test-2:
    container_name: test_2
    image: image
    networks:
      testing-net:
        ipv4_address: 172.28.1.2
  
  test-3:
    container_name: test_3
    image: image
    networks:
      testing-net:
        ipv4_address: 172.28.1.3

networks:
  testing-net:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
```

```yaml
version: '2'
services:
  mullvad:
    container_name: <name>
    image: <vpn_server_image>
    command: sleep infinity
    volumes:
      - "./openvpn:/etc/openvpn"
    networks:
      vpn:
        ipv4_address:172.20.0.1
    devices:
      - "/dev/net/tun:/dev/net/tun"
    privileged: true
    cap_add:
      - NET_ADMIN
networks:
  vpn:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.20.0.0/16
        - gateway: 172.20.0.1
```

## **Imágenes**

Podemos hacer que nuestro docker-compose cargue una imagen a partir de un dockerfile:
```yaml
version: '3'
services:
  web-1:
    container_name: web-1
    image: web-test
    # Indicamos que hay un dockerfile
    build: .
```
siendo el dockerfile:
```dockerfile
FROM centos
RUN mkdir /opt/dir
```

Si nuestro dockerfile se encuentra en otra carpeta, es posible especificar la ruta y el nombre de nuestro dockerfile:
```yaml
version: '3'
services:
  web-1:
    container_name: web-1
    image: web-test
    build:
      # Ubicacion
      context: ./docker
      # Nombre dockerfile
      dockerfile: dockerfile1
```

Podemos también modificar el `CMD` como lo hacíamos con los dockerfiles:
```yaml
version: '3'
services:
  web:
    container_name: web-1
    image: centos
    command: echo hello-world
```

Lanzamos el archivo y hacemos un log dentro del contenedor:
'''bash
$ docker-compose up -d
Creating network "mod-cmd_default" with the default driver
Creating web-1 ... done

$ docker logs web-1
hello-world
'''

## **Limitar recursos**

Podemos limitar los recurso de nuestro docker compose:
```yaml
version: '3'
services:
  web-2:
    container_name: nginx-2
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: '50M'
    image: nginx
```

## **Políticas de reinicio**

Existen cuatro tipos de política de reinicio:

- **no**

    Quiere decir que no existe renicio.

- **on-failure**

    Hay reinicio cunado se tiene un fallo.

- **unless-stopped**

    No se reinicia el contenedor a menos que se detenga específicamente o que el reinicio de docker lo detenga o lo reinicie.

- **always**

    Siempre se reinicia.

Para ver cómo funcionan, construimos el siguiente docker-compose:
```yaml
version: '3'
services:
  test:
    container_name: test
    image: restart-image
    build: .
```
El dockerfile desde el que se construye la imagen es:
```dockerfile
FROM centos
# Copio script
COPY start.sh /start.sh
# Le doy permiso de ejecución
RUN chmod +x /start.sh
# Ejecuto
CMD /start.sh
```
El script es el siguiente:
```bash
#!/bin/bash
echo "Estoy vivo"
sleep 5
echo "Estoy muerto"
```

Construimos la imagen:
```bash
$ docker-compose build
```

Lanzamos el contenedor:
```bash
$ docker-compose up -d
```

Por defecto, el contenedor nos aparece como no activo, ya que tiene como política de reinicio la tipo "none". Si añadimos la política de reinicio "always":
```yaml
version: '3'
services:
  test:
    container_name: test
    image: restart-image
    build: .
    restart: always
```
Podremos observar cómo ahora el contenedor se reinicia cada 5 segundos.