# Objetivo

La idea es configurar un contenedor con Jenkins, para poder montar una pipeline de CI/CD.

## Configurar Jenkins

- [services.jenkins.image] Ponemos la imagen que queremos utilizar, en este caso vamos a utilizar la imagen oficial de Jenkins.
- [services.jenkins.privileged] Le damos privilegios al contenedor.
- [services.jenkins.user] Le damos permisos de root.
- Establecer los puestos
  - [services.jenkins.ports] Establecemos los puertos que vamos a utilizar.
  - <destino>:<origen> En este caso vamos a utilizar el puerto 9080 para el 8080 y el 50000 para el 50000.
- [services.jenkins.container_name] Nombre del contenedor.
- [services.jenkins.volumes] Volumen donde se guardaran los datos.

```yaml
version: "3.8"
services:
  jenkins:
    image: jenkins/jenkins:lts
    privileged: true
    user: root
    ports:
      - 9080:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - jenkins_data:/var/jenkins_home
volumes:
  jenkins_data:
```

## Configurar el volumen

Lo indicamos que el directorio de jenkins_home se guarde en la carpeta **/Users/davidblancoparis/jenkins_home**.

```yaml
volumes:
  jenkins_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/davidblancoparis/data/jenkins_home
```

> Recordar que si se cambia hay que borrar el volumen y volver a crearlo.

## Plantilla

```yaml
version: "3.8"
services:
  jenkins:
    image: jenkins/jenkins:lts
    privileged: true
    user: root
    ports:
      - 9080:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - jenkins_data:/var/jenkins_home
volumes:
  jenkins_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /Users/davidblancoparis/data/jenkins_home
```
