# Insalacion del proxy de docker

La idea es instalar un proxy de docker para poder tener un cache de las imagenes que se descargan de internet, esto nos permitira tener un mejor rendimiento en la descarga de las imagenes y ademas nos permitira tener un control de las imagenes que se descargan en nuestra red.

url: https://earthly.dev/blog/private-docker-registry/

```ymal
version: "3.8"

services:
  registry:
    image: registry:latest
    ports:
      - "5001:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /registry-data
    volumes:
      - ./auth:/auth
      - ./registry-data:/registry-data


```

Con el siguiente comando creamos el archivo de password para el usuario que se va a autenticar en el registry

```bash
htpasswd -Bc registry.pasword adminuser
```

> Auth: en este directorio guardamos la contraseña.
