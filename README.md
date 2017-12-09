# docker-notes
My very own Docker notes.

### Table of Contents
**[Introducción](#introducción)**<br>
**[Instalación](#instalación)**<br>
**[Containers](#containers)**<br>
**[Ports in Containers](#ports-in-containers)**<br>
**[Images](#images)**<br>
**[Limpieza](#limpieza)**<br>
**[Ciclo de vida de un contenedor (Create/Start/Stop/Kill/RM)](#ciclo-de-vida-de-un-contenedor-createstartstopkillrm)**<br>
**[Dockerfiles](#dockerfiles)**<br>
**[Volumes](#volumes)**<br>
**[Networking](#networking)**<br>
**[Compose: Linkar Containers](#compose-linkar-containers)**<br>


# Introducción

* **Image:** An _image_ is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.

* **Container:**
  * A _container_ is a runtime instance of an image—what the image becomes in memory when actually executed.
  * It runs completely isolated from the host environment by default, only accessing host files and ports if configured to do so.
  * Containers **run apps natively** on the host machine’s kernel.
  * They have _better performance_ characteristics than virtual machines that only get virtual access to host resources through a hypervisor.
  * Containers can get native access, each one running in a discrete process, taking no more memory than any other executable.

# Instalación

Cuando hablamos de "instalar docker" nos estamos refiriendo a instalar "Docker Engine". Para ello, lo haremos siguiendo los pasos indicados en la [Documentación Oficial](https://docs.docker.com/engine/installation/).

# Containers
A continuación se muestran los comandos más habituales con "containers":

### `ps`

Muestra los contenedores en ejecución (running).

* Ver containers arrancados:                         
```shell
$ docker ps
```
* Ver "todo el historial" de containers arrancados:
```shell
$ docker ps -a
```

* Ver el "ID" (-q) del último (-l) container:
```shell
$ docker ps -l -q
```

### `exec`
Para acceder a la shell de un "container" que tuvieramos previamente arrancado:

Indicando su ID:
```shell
$ sudo docker exec -i -t 665b4a1e17b6 /bin/bash
```

o sino también a través de su nombre:
```shell
$ sudo docker exec -i -t loving_heisenberg /bin/bash
```

### `run`
Crea y arranca un "container".
Por defecto un contenedor se arranca, ejecuta el comando que le digamos y se para:
```shell
$ docker run busybox echo hello world
```

Podemos pasarle varias opciones al comando `run`, como por ejemplo:

* `run` **interactivo**:
```shell
$ docker run -t -i ubuntu:16.04 /bin/bash
```
  * **-h:** Le configuramos un hostname al "container".
  * **-t:** Asigna una TTY.
  * **-i:** Nos comunicamos con el "container" de manera interactiva.

  **Nota:** Al salir del modo interactivo (-i) el "container" se detendrá.

* `run` **Detached Mode**:

  Como ya sabemos, trás correr un contenedor de manera interactiva, este finaliza. Si se quieren hacer contenedores que corran servicios (por ejemplo, un servidor web) el comando es el siguiente:
  ```shell
  $ docker run -d -p 1234:1234 python:2.7 python -m SimpleHTTPServer 1234
  ```
  * **Explicación:** Esto ejecuta un servidor **Python** (SimpleHTTPServer module), en el puerto **1234**.
    * **-p 1234:1234** le indica a Docker que tiene que hacer un _port forwarding_ del contenedor hacia el puerto 1234 de la máquina host. Ahora podemos abrir un browser en la dirección http://localhost:1234.

    * **-d:** hace que el "container" corra en segundo plano. Esto nos permite ejecutar comandos sobre el mismo en cualquier momento mientras esté en ejecución. Por ejemplo:
    ```shell
    $ docker exec -ti <container-id> /bin/bash
    ```

      * Aquí simplemente se abre una **tty** en modo **interativo**. Podrían hacerse otras cosas como cambiar el _working directory_, definir variables de entorno, etc.


### `inspect`
Muestra detalles acerca de un contenedor:

* Info sobre un container:
```shell
$ docker inspect
```

* Dirección IP de container:
```shell
$ docker  inspect --format '{{ .NetworkSettings.IPAddress }}' <nombre_container>
```

### `logs`
* Mostrar logs sobre un container:
```shell
$ docker logs
```

### `stats`

* Estadísticas del container (CPU,MEM,etc.):
```shell
$ docker stats
```

### Ports in Containers
* Puertos publicos del container:                                        
```shell
$ docker port
```
* Publicar puerto 80 del container en un puerto random del Host:
```shell
$ docker run -p 80 nginx
```
* Publicar puerto 80 del container en el puerto 8080 del Host:
```shell
$ docker run -p 8080:80 nginx
```
* Publicar todos los puerto expuestos del container en puertos random del Host:
```shell
$ docker run -P nginx
```
* Listar todos los mapeos de los puertos de un container:
```shell
$ docker port <container_name>
```


# Images

* Ver listado de imagenes:
```shell
$ docker images
```

* Ver "todo el historial" de imágenes arrancadas:
```shell
$ docker images -a
```

* Borrar una imágen:
```shell
$ docker rmi <images_name>
```

* Para conocer el historial y "layers" que tiene una imagen:
```shell
$ docker history <image_name>
```

  ### Crear nuestras propias Images:

  Podemos crear imágenes diferentes maneras:

  A) `docker commit`: build an image from a container.

  Ejemplo:
  ```shell
   $ docker commit -m "Mensaje que queramos" -a "Nombre del que lo ha hecho" container-id NEW_NAME:TAG
   $ docker commit -m "MongoDB y Scrapy instalados" -a "Etxahun" 79869875807 etxahun/scrapy_mongodb:0.1
   ```
  B) `docker build`: create an image from a "Dockerfile" by executing the build steps given in the file.

  Dentro de un Dockerfile las "instructions" que podemos utilizar son las siguientes:

   * **FROM**: the base image for building the new docker image; provide "FROM scratch" if it is a base image itself.
   * **MAINTAINER**: the author of the Dockerfile and the email.
   * **RUN**: any OS command to build the image.
   * **CMD**: specify the command to be stated when the container is run; can be overriden by the explicit argument when providing docker run command.
   * **ADD**: copies files or directories from the host to the container in the given path.
   * **EXPOSE**: exposes the specified port to the host machine.


  Ejemplo:
  ```shell
  $ nano myimage/Dockerfile

  FROM ubuntu
  RUN echo "my first image" > /tmp/first.txt

  $ docker build -f myimage/Dockerfile   || o sino ||   docker build myimage
  Sending build context to Doker daemon 2.048 kB
  Step 1: FROM ubuntu
  ----> ac526a456ca4
  Step 2: RUN echo "my first image" > /temp/first.txt
  ----> Running in 18f62f47d2c8
  ----> 777f9424d24d
  Removing intermediate container 18f62f47d2c8
  Succesfully built 777f9424d24d

  $ docker images | grep 777f9424d24d

  <none>	<none>		777f9424d24d		4 minutes ago		125.2 MB

  $ docker run -it 777f9424d24d
  $ root@2dcd9d0caf6f:/#
  ```
  Podemos ponerle un nombre o "tagear" la imagen en el momento que estemos haciendo el "build":
  ```shell
  $ docker build <dirname> -t "<imagename>:<tagname>"
  ```
  Ejemplo:
  ```shell
  $ docker build myimage -t "myfirstimage:latest"
  ```

# Limpieza
### Containers:
* Borrar container:
```shell
$ docker rm <container_ID>
```

* Borrar container y sus volumenes asociados:
```shell
$ docker rm -v <container_ID>
```

* Borrar TODOS los containers:
```shell
$ docker rm $(docker ps -a -q)
```

### Images:
* Borrar imágenes:
```shell
$ docker rmi <container_ID>
```

* Borrar TODAS las imáganes:
```shell
$ docker rmi $(docker images -q)
```
* Listar imágenes <none> "colgadas":
```shell
$ docker images -f "dangling=true"
```

* Borrar imágenes <none> "colgadas":
```shell
$ docker rmi $(docker images -f "dangling=true" -q)
```

### Volumes:
* Borrar TODOS los "volume" que no se estén utilizando:
```shell
$ docker volume rm $(docker volumels -q)
```


# Ciclo de vida de un contenedor (Create/Start/Stop/Kill/RM)

Hasta ahora vimos cómo ejecutar un contenedor tanto en *foreground* como en *background* (detached). Ahora veremos cómo manejar el ciclo completo de vida de un contenedor. Docker provee de comandos como `create` , `start`, `stop`, `kill` , y `rm`. En todos ellos podría pasarse el argumento "-h" para ver las opciones disponibles.
Ejemplo:
```shell
$ docker create -h
```

Más arriba vimos cómo correr un contenedor en segundo plano (detached). Ahora veremos en el mismo ejemplo, pero con el comando `create`. La única diferencia es que ésta vez no especificaremos la opción "-d". Una vez preparado, necesitaremos lanzar el contenedor con `docker start`.

Ejemplo:
```shell
$ docker create -P --expose=8001 python:2.7 python -m SimpleHTTPServer 8001
  a842945e2414132011ae704b0c4a4184acc4016d199dfd4e7181c9b89092de13

$ docker ps -a
  CONTAINER ID IMAGE      COMMAND              CREATED       ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT 8 seconds ago ... fervent_hodgkin

$ docker start a842945e2414
  a842945e2414

$ docker ps
  CONTAINER ID IMAGE      COMMAND              ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT ... fervent_hodgkin
```

### Detener Containers:

Siguiendo el ejemplo, para **detener** el contenedor se puede ejecutar cualquiera de los siguientes comandos `kill` ó `stop`:
```shell
$ docker kill a842945e2414 (envía SIGKILL)
$ docker stop a842945e2414 (envía SIGTERM).
```

Asimismo, pueden reiniciarse (hacer un `docker stop a842945e2414` y luego un `docker start a842945e2414`):

```shell
$ docker restart a842945e2414
```

o destruirse:

```shell
$ docker rm a842945e2414
```

# Dockerfiles

# Volumes

# Networking

# Compose: Linkar containers

Cuando queramos "linkar" dos o más contenedores tendremos que establecer su relación en un fichero YAML. A continuación se muestra un ejemplo de un fichero que "linka" un container "Web" (Wordpress) y uno de base de datos "MySQL":

* Contenido del fichero **ejemplo.yml**:

  ```YAML
  web:
     image: wordpress
     links:
       - mysql
     environment:
       - WORDPRESS_DB_PASSWORD=sample
     ports:
       - "127.0.0.3:8080:80"
  mysql:
  image: mysql:latest
  environment:
     - MYSQL_ROOT_PASSWORD=sample
     - MYSQL_DATABASE=wordpress

  ```
  Y para ejecutarlo, estando en el mismo directorio donde está el fichero **ejemplo.yml**, haremos lo siguiente:

  ```shell
  $ docker-compose up
  ```
  Y para comprobar que todo ha ido bien, abriremos la url http://127.0.0.3:8080 para aceder a la página de Wordpress.
