# Docker

## Comandos esenciales

`docker pull <nombre de imagen>`: Descarga una imagen desde Docker Hub o un registro personalizado.

`docker run <nombre de imagen>`: Crea y ejecuta un contenedor a partir de una imagen.

`docker ps`: Muestra los contenedores en ejecución.

`docker stop <ID de contenedor>`: Detiene un contenedor en ejecución.

`docker rm <ID de contenedor>`: Elimina un contenedor.

`docker images`: Muestra las imágenes disponibles en el registro local.

`docker rmi <nombre de imagen>`: Elimina una imagen local.

`docker exec -it <ID de contenedor> <comando>`: Ejecuta un comando en un contenedor en ejecución.

`docker logs <ID de contenedor>`: Muestra los registros de un contenedor.

## docker pull

```
docker pull mysql
```

El comando `docker pull mysql` se usa para descargar la imagen oficial de MySQL desde Docker Hub al registro local de tu máquina. Esto es útil para crear contenedores de MySQL sin necesidad de configurarlo desde cero.

## docker images

El comando `docker images` se utiliza para listar todas las imágenes de Docker que están disponibles localmente en tu sistema. Al ejecutarlo, verás una tabla con información detallada sobre cada imagen, incluyendo:

- **REPOSITORY**: El nombre del repositorio de la imagen (por ejemplo, `mysql`).
- **TAG**: La etiqueta de la imagen, que generalmente indica la versión (por ejemplo, `latest`).
- **IMAGE ID**: Un identificador único para la imagen.
- **CREATED**: La fecha en que se creó la imagen.
- **SIZE**: El tamaño de la imagen en disco.

```
C:\>docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        latest    fd8d1b4e287c   2 weeks ago   826MB
```

## Acceder docker Desde CLI

```
docker login -u User
```

```
Token
```

## docker run

El comando `docker run` se utiliza para crear y ejecutar un contenedor a partir de una imagen de Docker. Es uno de los comandos más versátiles y permite configurar múltiples aspectos del contenedor.

```
docker run -e MYSQL_ROOT_PASSWORD=123456 mysql
```

Para ejecutar el contenedor de forma desacoplada del terminal ejecute el comando siguiente:

```
docker run -d -e MYSQL_ROOT_PASSWORD=123456 mysql
```

## Eliminar contenedor

1. Visualice los contenedores con el comando docker ps -a

2. Ejecute el comando docker rm IdContenedor

   ```
   C:\>docker rm c76651bc274d
   c76651bc274d
   ```

## Eliminar Imagenes

El comando `docker rmi` se utiliza para eliminar una o más imágenes de Docker de tu sistema. Este comando es útil para liberar espacio en disco eliminando imágenes que ya no necesitas.

```
docker rmi <nombre de la imagen> o <ID de la imagen>
```

```
C:\>docker rmi mysql
Untagged: mysql:latest
Deleted: sha256:fd8d1b4e287c49e1e35eb5a103f337111947662130eb8a3e6c3e823813f47f7d
```

Ejecute el comando **docker images** para visualizar las imagenes disponibles

```
C:\>docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

## Conectarse Contenedor Postgres

Primero verifica el nombre del contenedor:

```
docker ps
```

Deberías ver algo como:

```
CONTAINER ID   IMAGE         NAMES
abc123         postgres:16   postgres_db
```

Si tu contenedor se llama `postgres_db`, para entrar a su terminal ejecuta:

```
docker exec -it postgres_db bash
```

Desde ahí puedes entrar a PostgreSQL con:

```
psql -U bkseducate -d bkddb
```

# Cómo abrirlo en Visual Studio Code

Instala en VS Code la extensión:

```
Dev Containers
```

de Microsoft.

Luego abre la carpeta:

```
postgres-container/
```

y usa:

```
Ctrl + Shift + P
```

en Windows/Linux, o:

```
Cmd + Shift + P
```

en macOS.

Busca:

```
Dev Containers: Reopen in Container
```

VS Code hará automáticamente:

```
leer devcontainer.json
        ↓
leer docker-compose.yml
        ↓
construir workspace
        ↓
levantar PostgreSQL
        ↓
levantar pgAdmin
        ↓
abrir /workspace
```

Dentro del terminal de VS Code puedes comprobar:

```
psql --version
```

y conectarte directamente:

```bash
psql \
  -h postgres_db \
  -p 5432 \
  -U bkseducate \
  -d bkddb
```

Te pedirá:

```
Password:
```

y colocas:

```
bkseducate2026
```

Observa nuevamente la regla:

```
Desde VS Code dentro del contenedor:
postgres_db:5432
```

Pero desde tu Mac:

```
localhost:5433
```

pgAdmin quedaría accesible desde el navegador en:

```
http://localhost:8081
```



Abre pgAdmin en:

```
http://localhost:8081
```

Inicia sesión con:

```
Email: admin@example.com
Password: SuperBks123!
```

Luego ve a:

```
Servers
→ Register
→ Server
```

En la pestaña **General**, coloca:

```
Name:
PostgreSQL Local
```

Ese nombre puede ser cualquiera; solo sirve para identificar la conexión dentro de pgAdmin.

En la pestaña **Connection**, usa exactamente estos valores:

```
Host name/address:
postgres_db

Port:
5432

Maintenance database:
bkddb

Username:
bkseducate

Password:
bkseducate2026
```

Activa también:

```
Save password
```

La razón de usar:

```
postgres_db
```

es que pgAdmin está ejecutándose dentro de Docker. Para él, PostgreSQL no está en `localhost`, sino en otro contenedor accesible por el nombre del servicio.