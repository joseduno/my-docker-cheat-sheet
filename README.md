# Guia docker

Install (linux): https://docs.docker.com/engine/install/ubuntu/

## Imagenes
Pararrer contenedor a partde una imagen y mantenerlo corriendo siempre (con el paráro -d):
```
sudo docker run -d <imagen>:<tag>
```
el comando anterior descarga la imagen en caso de no existir (docker pull), si existiese la corre. Parastar las imagenes descargadas:
```
sudo docker images
```
Esto no muestra los contenedores activos

## Contenedores
Parastar contenedores corriendo:
```
sudo docker ps
```
Paraiciar un contenedor previamente cerrado por su CONTAINER ID (se puede ver ejecutando docker ps -a)
```
sudo docker start <CONTAINER ID>
```
También se puede detener un contenedor corriendo con:
```
sudo docker stop <CONTAINER ID>
```
Parar el log de un contenerdor concreto por CONTAINER ID O NAMES (se puede ver ejecutando docker ps -a)
```
sudo docker logs [<CONTAINER ID>, <NAME>]
```
con el paráro -f se puede ver en tiempo real el contenido del log
```
sudo docker logs -f [<CONTAINER ID>, <NAME>]
```
Paraecutar terminal dentro de un contenedor que está corriendo se utiliza
```
sudo docker exec -it <CONTAINER ID> sh
```

## Dockerfile
Archivo para crear imagenes
- **FROM:** siempre es la primera linea e indica la imagen padre a partde la cual se creará la nueva imagen
- **WORKDIR:** sobre esta dirección dentro de la imagen se ejecutaran los comandos a menos que se especifique lo contrario
- **COPY:** copiar archivos desde el host hacia la imagen, la ruta "." a la izquierda apunta hacia el directorio del host donde se encuentra el Dockerfile, y la ruta "." a la derecha apunta hacia el WORKDIR de la imagen
- **RUN:** ejecuta el comando en la carpeta definida por WORKDIR
- **CMD:** comando que se ejecutara al final, tambien se puede usar **ENTRYPOINT** pararizar la ejecución de este comando al correr el contenedor basado en la imagen que crea el Dockerfile
- 

Paranstruir imagen a partdel Dockerfile:
```
sudo docker build -t <imagen>:<tag> .
```
La ruta "." apunta hacia donde se encuentre el Dockerfile, luego ejecutando docker run se creará contenedor a partir de la imagen creada.

## Compartiendo imagenes
Tener precaucion, compartir imágenes es gratuito pero públicas

Para loguearte en docker hub
```
sudo docker login
```
 Los nombres de las imagenes deben seguir el formato **usuario/nombre:tag** para que sea gratuito, para cambiar el tag de una imagen:
```
docker tag <IMAGE ID> <usuario/imagen>:<tag>
```
IMAGE ID se encuentra listando las imágenes con el comando **docker images**

Para publicar la imagen
```
sudo docker push <usuario/imagen>:<tag>
```

## Sin docker-compose
Si la imagen contiene alguna aplicación, será necesario exponer los puertos para uso desde el host, esto se hace de la siguiente forma:
```
sudo docker run -d -p <puertoHost>:<puertoContenedor> <imagen>:<tag>
```
Montando volúmenes se sincronizan los cambios de forma bidireccional entre el host y el contenedor
```
sudo docker run -d -v <ruta/al/directorio/host>:<ruta/al/directorio/contenedor> -p <puertoHost>:<puertoContenedor> <imagen>:<tag>
```
Si hay volúmenes y se han hecho cambios en el host, reejecutando el comando _**sudo docker build -t <imagen>:<tag> .**_ se construye una nueva imagen con los cambios realizados, El comando _**COPY**_ detecta que hay cambios y copia lo nuevo en la nueva imagen (esto es similar a hacer un git push al final de una sesión de trabajo)

Para pasar variables de entorno como parámetros se utiliza -e:
```
sudo docker run -d \
-v <./ruta/al/directorio/host>:<./ruta/al/directorio/contenedor> \
-e <VARIABLE>=<VALOR> \
-p <puertoHost>:<puertoContenedor> \
<imagen>:<tag>
```

## Con docker-compose
El archivo docker-compose.yaml, permite correr varios contenedores (en el archivo se describen como servicios) y especificar las particularidades de cada uno como puertos y variables de entornos. Además crea una red donde conviven cada uno de los contenedores especificados sin necesidad de crearla explicitamente.

_**Nota:** el docker-compose crea un network alias por cada servicio definido con el mismo nombre del servicio, esto es util para establecer las comunicaciones entre contenedores_

##### Estructura del archivo #####
**version**: "3.7"
**services**:
**app**:
image: <imagen>:<tag>
ports:
- <puertoHost>:<puertoContenedor>
environment:
VARIABLE1: VALOR1
VARIABLE2: VALOR2
**mysql**: <imagen>:<tag>
image:
volumes:
- <./ruta/al/directorio/host>:<./ruta/al/directorio/contenedor>
environment:
VARIABLE1: VALOR1
VARIABLE2: VALOR2

Para ejecutar los contenedores definidos en el archivo docker-compose.yaml
```
sudo docker-compose up -d
```
Para detener los contenedores especificados en el archivo docker-compose.yaml
```
sudo docker-compose down
```