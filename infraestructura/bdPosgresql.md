# Objetivo

- Ver como podemos crear una BD local de posgresql.

* Donde guardar los datos de postgresql
* Inicializacion de la BD.

> Recursos (https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/ > https://kinsta.com/es/blog/volumenes-docker-compose/
> )

## creamos el fichero de docker-compose (compose.yaml)

- [version] Poner las version del archivo (En este caso la 3.9)
- [services.db]Vamos a definir los diferentes contenedores que colgaran de la etiqueta services, en este caso vamos a crear un que es la db (Que va a ser la bd)
- [services.db.imagen]: Establecemos la imagen que nos interesa crear en esta caso la oficial sera postgres
- [services.db.restart]: Le vamos a indicar que si se cae se vuelva a levantar.

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    restart: "always"
```

## Vamos a parametrizar la BD

Las variables se establecen en el environment, en este caso vamos a establecer las siguietnes variables:

- POSTGRES_USER: Usuario de la BD
- POSTGRES_PASSWORD: Contraseña del usuario
- POSTGRES_DB: Nombre de la BD

Vamos a establecer el puerto en el 5432, le indicamos que el puerto 5432 va a salir por 5432.

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    restart: "always"
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "postgres"
    ports:
      - "5432:5432"
```

> Los datos son de ejemplo

## Añadir los volumenes

Vamos a añadir donde queremos que se guarden los datos, para este caso vamos a utilizar la variable PGDATA.

- [services.db.volumes]: Vamos a establecer el volumen que queremos que se guarde en el contenedor, en este caso vamos a guardar en la carpeta **pgdata:/var/lib/postgresql/data**.

Hay que declararla

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    restart: "always"
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - "pgdata:/var/lib/postgresql/data"
volumes:
  pgdata:
```

> PGDATA: Es el directorio donde se guardan los datos de la BD. (PG_VERSION (version de la bd) y base subdirectorio con los datos )

> Ahora nos queda ver como vamos a configurar el volumen, para indicarle donde queremos que se guarde.

## Configurar el volumen

En nuestro ejemplo nos interesa, que los datos se guarden en una carpeta local (/Users/davidblancoparis/bd/localpg), para lo cual vamos a configurar las siguietnes variables:

- driver: Tipo de driver que vamos a utilizar, en este caso vamos a utilizar local.
- driver_opts: Opciones que vamos a utilizar, en este caso vamos a indicarle donde queremos que se guarde.
- device: Directorio donde queremos que se guarde (/Users/davidblancoparis/bd/localpg).
- o: (OPT) es la opción de driver. (bind)
- type: none
  > Este es el ejemplo para configurar un directorio, al parecer estan relaciando con el parametro opts de docker y se vincula al SO.

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    restart: "always"
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - "pgdata:/var/lib/postgresql/data"
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/davidblancoparis/bd/localpg
```

> Si se cambia la configuración del volumen, hay que eliminar el volumen y volver a crearlo.

## Comandos para trabajar con los volumenes:

- docker volume ls: Listar los volumenes.
- docker volume rm <nombre_volumen>: Eliminar un volumen.
- docker volume inspect <nombre_volumen>: Inspeccionar un volumen.

## Configurar los recursos

En este caso lo que vamos a ver es como configurar la memoria.

Se establece en la configuracion del servicio, en este caso vamos a establecer la memoria en 512MB.

- [services.db.deploy.resources.limits.memory]: Establecemos la memoria en 512MB.

```yaml
deploy:
  resources:
    limits:
      memory: 256M
```

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    deploy:
      resources:
        limits:
          memory: 256M
    restart: "always"
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - "pgdata:/var/lib/postgresql/data"
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/davidblancoparis/bd/localpg
```

## Añadir un healthcheck

Es un servicio que sirve para saber si el servidor esta levantado o caido, para lo cual usaremos el servicio pg_isready que nos dira si el servidor esta levantado o no.

```yaml
version: "3.9"
services:
  db:
    image: "postgres"
    restart: "always"
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "postgres"
    ports:
      - "5432:5432"
    volumes:
      - "pgdata:/Users/davidblancoparis/bd/localpg"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
        interval: "10s"
      timeout: "5s"
        retries: "5"
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/davidblancoparis/bd/localpg
```

## Variables de entorno:

- POSTGRES_USER: Usuario de la BD
- POSTGRES_PASSWORD: Contraseña del usuario
- POSTGRES_DB: Nombre de la BD
- PGDATA: Directorio donde se guardan los datos de la BD.
- POSTGRES_INITDB_ARGS: Argumentos que se le pasan al comando initdb.
- postgres_initdb: Comando que se ejecuta al inicializar la BD.
- POSTGRES_INITDB_WALDIR: Directorio del registro de transacciones.
- POSTGRES_HOST_AUTH_METHOD: Metodo de autenticacion que se va a utilizar.
