# **Networking**

## **Estado y nombre de la red**
Todo contenedor, cuando se levanta, se conecta automáticamente.
Por ejemplo, levantamos el siguiente contenedor:
```bash
$ docker run -dti --name centos centos
```
Veámos cuál es su dirección IP:
```bash
$ docker container inspect centos | findstr "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```
Podemos consultar las redes disponibles:
```bash
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f1977a037637   bridge    bridge    local
a25d0070553e   host      host      local
e956fde01cc0   none      null      local
```
incluso cada tipo de red:
```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "f1977a037637c9437ad81a05dabbafe7e041dc6dff9a53020cb01b45d779cd64",
        "Created": "2022-09-19T15:48:28.3658677Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "419ba840efafb53f00207a1c61afb574a64b536969c0dc73ff22246e53e1b908": {
                "Name": "centos",
                "EndpointID": "9b53897fbbb4ca48d3caeb41038c0147e9bc3c91df1be69a979f9fb00eade0d2",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
y filtrar nuestra consulta:
```bash
$ docker network inspect bridge | findstr "Gateway"
                    "Gateway": "172.17.0.1"

$ docker network inspect bridge | findstr "Subnet" 
                    "Subnet": "172.17.0.0/16",
```

Si queremos desconectar nuestro contenedor de la red:
```bash
$ docker network disconnect [NETWORK] [NOMBRE_CONT]
```

## **Conexión entre contenedores de la misma red**

Si creamos otro contenedor en la misma red:
```bash
$ docker run -dti --name centos2 centos
```
podremos comprobar que el IPAdress es diferente al contenedor anterior:
```bash
$ docker container inspect centos2 | findstr "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
```
Dado que ambos contenedores están conectados a la red bridge, puedo establecer una comunicación entre ellos:
```bash
$ docker exec centos2 bash -c "ping 172.17.0.2"
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.123 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.110 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.105 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.090 ms
64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.089 ms
64 bytes from 172.17.0.2: icmp_seq=7 ttl=64 time=0.060 ms
64 bytes from 172.17.0.2: icmp_seq=8 ttl=64 time=0.084 ms
64 bytes from 172.17.0.2: icmp_seq=9 ttl=64 time=0.067 ms
64 bytes from 172.17.0.2: icmp_seq=10 ttl=64 time=0.100 ms
64 bytes from 172.17.0.2: icmp_seq=11 ttl=64 time=0.103 ms
```

## **Mapeo de puertos**

Creamos el siguiente contenedor:
```bash
$ docker run -d --name nginx -p 9090:80 nginx
```
Consultamos el puerto:
```bash
$ docker container inspect nginx | findstr "./tcp"
                "80/tcp": [
                "80/tcp": {}
                "80/tcp": [
```
tambien se puede consultar del siguiente modo:
```bash
$ docker port nginx
80/tcp -> 0.0.0.0:9090
```
Si quiero emparejar los puertos automáticamente:
```bash
$ docker run -d --name nginx --publish-all  nginx
```

## **Creación de redes**
Para crear una nueva red ejecutamos el comando:
```bash
$ docker network create [NOMBRE]
```
por ejemplo:
```bash
$ docker network create docker-test-network
```

Ahora creamos un contenedor conectado a esa red:
```bash
$ docker run -d --name docker-test-net --network docker-test-network centos
```

Podemos especificar cualquier característica de nuestra red al crearla:
```bash
$ docker network create -d bridge --subnet 172.124.10.0/24 --gateway 172.124.10.1 [NOMBRE_NET]
```

## **Comunicación entre contenedores en red**

Cuando dos contenedores comparten la misma red, pueden llamarse el uno al otro simplemente por sus nombres. Para ello, creamos una nueva red:
'''bash
$ docker network create net-test
'''
y levantamos dos contenedores conectados a ella:
```bash
$ docker run -d --network net-test --name test1 -ti centos
$ docker run -d --network net-test --name test2 -ti centos
```
Podríamos establecer una conexión entre ambos contenedores a través de sus direcciones IP, pero dado que están conectados a la misma red, basta con llamarlos por su nombre del siguiente modo:
```bash
$ docker exec test1 bash -c "ping test2"
PING test2 (172.18.0.3) 56(84) bytes of data.
64 bytes from test2.net-test (172.18.0.3): icmp_seq=1 ttl=64 time=0.105 ms
64 bytes from test2.net-test (172.18.0.3): icmp_seq=2 ttl=64 time=0.109 ms
64 bytes from test2.net-test (172.18.0.3): icmp_seq=3 ttl=64 time=0.105 ms
$ docker exec test2 bash -c "ping test1"
PING test1 (172.18.0.2) 56(84) bytes of data.
64 bytes from test1.net-test (172.18.0.2): icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from test1.net-test (172.18.0.2): icmp_seq=2 ttl=64 time=0.086 ms
64 bytes from test1.net-test (172.18.0.2): icmp_seq=3 ttl=64 time=0.063 ms
```

Otro aspecto interesante es que un contenedor puede tener dos endpoints diferentes. Para estudiar ésto, creamos dos nuevas redes:
```bash
$ docker network create net-a
$ docker network create net-b
```
y dos contenedores, cada uno de ellos conectado a una de las redes anteriores:
```bash
$ docker run -dti --name cont-a --network net-a centos
$ docker run -dti --name cont-b --network net-b centos
```

Ahora conectamos cont-a a la red net-b:
```bash
$ docker network connect net-b cont-a
```
Si inspeccionamos cont-a podremos ver cómo está conectado a ambas redes:
```bash
"Networks": {
                "net-a": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "befcc0195072"
                    ],
                    "NetworkID": "520e8b5abad90d954adcf32bb8846c690e853fe4ba9669dc2d69f72782d7ae33",
                    "EndpointID": "4837eef13c5bb19240d768597a0e9166b03a899eb3403774c0c475c74a0cb427",
                    "Gateway": "172.19.0.1",
                    "IPAddress": "172.19.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:13:00:02",
                    "DriverOpts": null
                },
                "net-b": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "befcc0195072"
                    ],
                    "NetworkID": "7dfc6ce245c3afe6b000ce3bc07ef14cc36217ca276e5f949f20b3309c67ff33",
                    "EndpointID": "b811de1b440f62a7d3ca2fa0985e08c1dda1a8a3c34f930f8e11982f107ae63b",
                    "Gateway": "172.20.0.1",
                    "IPAddress": "172.20.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:14:00:03",
                    "DriverOpts": {}
                }
            }
```
De éste modo, ahora el cont-a puede conectarse con el cont-b, ya que están en la misma red:
```bash
$ docker exec -ti cont-a bash -c "ping cont-b"
PING cont-b (172.20.0.2) 56(84) bytes of data.
64 bytes from cont-b.net-b (172.20.0.2): icmp_seq=1 ttl=64 time=0.091 ms
64 bytes from cont-b.net-b (172.20.0.2): icmp_seq=2 ttl=64 time=0.110 ms
64 bytes from cont-b.net-b (172.20.0.2): icmp_seq=3 ttl=64 time=0.103 ms
```
Además, si ahora creamos otro contenedor:
```bash
$ docker run -dti --name cont-c --network net-a centos
```
éste podrá conectarse al cont-a por la red net-a:
```bash
$ docker exec -ti cont-a bash -c "ping cont-c"        
PING cont-c (172.19.0.3) 56(84) bytes of data.
64 bytes from cont-c.net-a (172.19.0.3): icmp_seq=1 ttl=64 time=0.090 ms
64 bytes from cont-c.net-a (172.19.0.3): icmp_seq=2 ttl=64 time=0.076 ms
64 bytes from cont-c.net-a (172.19.0.3): icmp_seq=3 ttl=64 time=0.108 ms
```
Podemos asignarle una IP a un contenedor. Para ello, creamos la siguiente red:
```bash
$ docker network create --subnet 172.128.10.0/24 --gateway 172.128.10.1 -d bridge my-net
```
y ahora creamos un nuevo contenedor con una IP asignada:
```bash
$ docker run -d --network my-net --ip 172.128.10.50 --name nginx-v1 centos
```

## **Tipos de drivers para redes**

* **Bridge**
    - Es el controlador por defecto. Posee la configuración básica para conectar dos contenedores entre sí, siendo la mejor solución para conectar dos contenedores en el mismo host.

* **Host**
    - Elimina el aislamiento entre el controlador y el anfitrión, de modo que es posible acceder externamente.

* **Overlay**
    - Permite conectar diferentes contenedores en diferentes nodos.

* **Macvilla**
    - Permite asignar una dirección MAC a un contenedor.

* **None**
    - Inhabilita todas las redes, por si no queremos que nuestro contenedor esté conectado a nada.