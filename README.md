# Instalación automática de Wordpress con seguridad.

## Creación de los respectivos contenedores para la instalación de wordpress.

### ¿Qué es docker?
Docker es una plataforma de código abierto diseñada para facilitar la creación, implementación y ejecución de aplicaciones en contenedores. Un contenedor se puede definir como un entorno ligero, aislado y portable, que contiene todo lo necesario (código fuente, dependencias, etc.) para ejecutar una aplicación, un contenedor suele tener un único procesos en ejecución, aunque es posible tener varios. Una de las ventajas que aporta el uso de contenedores es que garantiza que una aplicación se ejecute de la misma manera en cualquier entorno.

Comandos importantes de docker:
```
Trabajamos con un fichero llamado docker-compose.yml
-Para cargar dicho fichero usamos el comando:
docker-compose up

- Para ver el listado de las imagenes
docker images

- Para borrar las imagenes:
docker rmi -f <id_imagen>

- Para listar los volumenes 
docker volume ls

- Para borrar las volumenes:
docker-compose down -v

- Para ver el listado de los contenedores incluso los que se encuentran en segundo plano:
docker ps -a

- Para borrar los contenedores
docker rm -f <docker id>

```

### Trabajamos con un fichero docker-compose.yml
Por tanto lo vamos a explicar por parte.

#### Comenzamos con la parte de Wordpress.
```
version: '3'

services:
  wordpress:
    image: bitnami/wordpress:6.4.3
    environment: 
      - WORDPRESS_DATABASE_HOST=mysql
      - WORDPRESS_DATABASE_NAME=${WORDPRESS_DATABASE_NAME}
      - WORDPRESS_DATABASE_USER=${WORDPRESS_DATABASE_USER}
      - WORDPRESS_DATABASE_PASSWORD=${WORDPRESS_DATABASE_PASSWORD}
      - WORDPRESS_BLOG_NAME=${WORDPRESS_BLOG_NAME}
      - WORDPRESS_USERNAME=${WORDPRESS_USERNAME}
      - WORDPRESS_PASSWORD=${WORDPRESS_PASSWORD}
      - WORDPRESS_EMAIL=${WORDPRESS_EMAIL}
    volumes: 
      - wordpress_data:/bitnami/wordpress
    depends_on:
      - mysql
    restart: always
    networks:
      - frontend-network
      - backend-network
```

Que tenemos aquí...
La versión, la cual usa la versión de docker-compose: ``3``

Seguidamente comenzamos con el servicio de wordpress que contiene estas ips para la instalación.

Sacando la imagen de docker hub, usando la instalacion de wordpress de bitnami tenemos que trabajar con las etiquetas que nos ofrece esa ofrece esa imagen de wordpress, es necesario establecer la version en concreto con la que vamos a trabajar ``6.4.3``

En environment visualizamos las variables que contiene información sobre la instalación automatizada de wordpress.
Estas variables, para se exactos:

```
DNS_DOMAIN_SECURE=ansible-frontend.ddns.net

WORDPRESS_DATABASE_HOST=mysql
WORDPRESS_DATABASE_NAME=wordpress
WORDPRESS_DATABASE_USER=wp_user
WORDPRESS_DATABASE_PASSWORD=wp_pass

MYSQL_ROOT_PASSWORD=root

WORDPRESS_BLOG_NAME="Sitio web de wordpress"

#WORDPRESS_PLUGINS=hidelogin

WORDPRESS_USERNAME=admin
WORDPRESS_PASSWORD=admin
WORDPRESS_EMAIL=demo@demo.es
```
Necesarios para la automatización de la instalación.

Se emplea tambien un volumen con nombre, para mantener la persistencia de datos, y que no se borren cuando se pare el contenedor.

```
    depends_on:
      - mysql
    restart: always
    networks:
      - frontend-network
      - backend-network
```

Docker-compose, funciona con etiquetas, no hace falta poner ips este fichero ya creará las redes con la etiqueta frontend-network y backend-network, tambien hay que tener en cuenta que depende de que mysql este en funcionamiento.

#### Comenzamos la parte de la instalación de mysql
```
  mysql:
    image: mysql:8.0
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${WORDPRESS_DATABASE_NAME}
      - MYSQL_USER=${WORDPRESS_DATABASE_USER}
      - MYSQL_PASSWORD=${WORDPRESS_DATABASE_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    restart: always
    networks:
      - backend-network
```

#### Comenzamos la parte de instalación de phpmyadmin.
```   
  phpmyadmin:
    image: phpmyadmin:5.2.1
    ports:
      - 8080:80
    environment: 
      - PMA_HOST=mysql
    restart: always
    networks:
      - frontend-network
      - backend-network
```

#### Comenzamos la parte de instalación de https en wordpress, para securizarlo.
```
  https-portal:
    image: steveltn/https-portal:1
    ports:
      - 80:80
      - 443:443
    restart: always
    environment:
      DOMAINS: '${DNS_DOMAIN_SECURE} -> http://wordpress:8080'
      STAGE: 'production' # Don't use production until staging works
      # FORCE_RENEW: 'true'
    networks:
      - frontend-network
```

#### Parte de los volumenes y redes creadas durante la creación del fichero docker-compose.yml
```
volumes: 
  mysql_data:
  wordpress_data:

networks:
  frontend-network:
  backend-network:
```

## Comprobación de que funciona el fichero.