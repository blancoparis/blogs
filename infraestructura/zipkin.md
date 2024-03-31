# Zipkin

Es una herraminta de trazabilidad distribuida, que permite monitorear y depurar microservicios. Es una herramienta de código abierto que se utiliza para rastrear solicitudes a través de microservicios. Zipkin es una herramienta de trazabilidad distribuida que se utiliza para monitorear y depurar microservicios. Zipkin es una herramienta de código abierto que se utiliza para rastrear solicitudes a través de microservicios. Zipkin es una herramienta de trazabilidad distribuida que se utiliza para monitorear y depurar microservicios. Zipkin es una herramienta de código abierto que se utiliza para rastrear solicitudes a través de microservicios.

## Endpoint

Vamos a ver los endpoint importantes de zipkin:

- Los que corresponde al servidor:
  / - UI
  /config.json - Configuration for the UI
  /api/v2 - API
  /health - Returns 200 status if OK
  /info - Provides the version of the running instance
  /metrics - Includes collector metrics broken down by transport type
  /prometheus - Prometheus scrape endpoint
  > Para mas información lo podemos ver en zipkin-server (https://github.com/openzipkin/zipkin/blob/master/zipkin-server/README.md)
- Los correspondientes a su api
  https://zipkin.io/zipkin-api/#/

## Construir el primer servidor de zipkin

Para esto hemos tenido que ir al proyecto de zipkip (https://github.com/openzipkin/zipkin/blob/master/README.md), y vamos hacer una imagen que se apoye en mysql (Ojo, la version de mysql esta obsoleta), pero para empezar me parece mas facil para realizar pruebas.

Las alternativas son:

- Memoria: En este caso perdemos la persistencia.
- ElasticSearch: Es una base de datos no relacional, que nos permite tener persistencia.
- cassandra: Es una base de datos no relacional, que nos permite tener persistencia.

> Podemos construir un zipkin con la persistencias en memoria y sin nada mas, pero creo que es mas interesante montar una arquitectura con persistencia.

### Mysql

En este caso vamos a utilizar los siguientes componentes:

- storage: Que sera una BD mysql.
- zipkin: Que sera el servidor de zipkin, tenemos que usar el (full, ya que el slim no tiene soporte a mysql).
- zipkin-dependencies: Es una herramienta que recopila trazas de conexion entre diferentes servicios y las almacena en una BD.

Los servicioes es necesario que esta levantada correctamente la BD de mysql y chequeda su salud, par alo cual vamos a usar la siguiente etiqueta:

```yaml
depends_on:
  storage:
    condition: service_healthy
```

#### Docker-compose

A continuación vamos a ver el docker-compose.yml, para montar la el zipkin

```yaml
version: "3.8"

services:
  storage:
    image: ghcr.io/openzipkin/zipkin-mysql:${TAG:-latest}
    container_name: mysql
    ports:
      - 3306:3306
  # The zipkin process services the UI, and also exposes a POST endpoint that
  # instrumentation can send trace data to.
  zipkin:
    image: ghcr.io/openzipkin/zipkin:${TAG:-latest}
    container_name: zipkin
    # Environment settings are defined here https://github.com/openzipkin/zipkin/blob/master/zipkin-server/README.md#environment-variables
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_HOST=storage
      # Add the baked-in username and password for the zipkin-mysql image
      - MYSQL_USER=zipkin
      - MYSQL_PASS=zipkin
    ports:
      # Port used for the Zipkin UI and HTTP Api
      - 9411:9411
    # Uncomment to enable debug logging
    # command: --logging.level.zipkin2=DEBUG
    depends_on:
      storage:
        condition: service_healthy
  dependencies:
    image: ghcr.io/openzipkin/zipkin-dependencies
    container_name: dependencies
    user: root
    entrypoint: /usr/sbin/crond -f
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_HOST=storage
      # Add the baked-in username and password for the zipkin-mysql image
      - MYSQL_USER=zipkin
      - MYSQL_PASS=zipkin
    depends_on:
      storage:
        condition: service_healthy
```

> Para obtener este zipkin nos hemos apoyado en el siguiente ejemplos (https://github.com/openzipkin/zipkin/blob/master/docker/README.md), lo hemos modificado para que este en un solo fichero y no ser un conjunto de multiples ficheros y verlo mas claro.

#### Arrancarlo

En este caso en el directorio, solo necesitamos el fichero docker-compose.yml, y ejecutamos el siguiente comando:

```bash
 docker compose up -d
```

> Ponemos las -d para que sea en segundo plano.

### Mysql + prometheus + grafana

Ahora vamos a ver como configurarlo para integrarlo con prometheus y grafana.

Para este caso tenemos que crear los ficheros de configuracion:

- create-datasource-and-dashboard.sh
- prometheus.yml

> Esto lo he sacado de los ejemplos de zipkin.

create-datasource-and-dashboard.sh

```bash
#!/bin/sh
#
# Copyright The OpenZipkin Authors
# SPDX-License-Identifier: Apache-2.0
#

set -xeuo pipefail

if ! curl --retry 5 --retry-connrefused --retry-delay 0 -sf http://grafana:3000/api/datasources/name/prom; then
    curl -sf -X POST -H "Content-Type: application/json" \
         --data-binary '{"name":"prom","type":"prometheus","url":"http://prometheus:9090","access":"proxy","isDefault":true}' \
         http://grafana:3000/api/datasources
fi

dashboard_id=1598
last_revision=$(curl -sf https://grafana.com/api/dashboards/${dashboard_id}/revisions | grep '"revision":' | sed 's/ *"revision": \([0-9]*\),/\1/' | sort -n | tail -1)

echo '{"dashboard": ' > data.json
curl -s https://grafana.com/api/dashboards/${dashboard_id}/revisions/${last_revision}/download >> data.json
echo ', "inputs": [{"name": "DS_PROMETHEUS", "pluginId": "prometheus", "type": "datasource", "value": "prom"}], "overwrite": false}' >> data.json
curl --retry-connrefused --retry 5 --retry-delay 0 -sf \
     -X POST -H "Content-Type: application/json" \
     --data-binary @data.json \
     http://grafana:3000/api/dashboards/import

```

prometheus.yml

```yaml
#
# Copyright The OpenZipkin Authors
# SPDX-License-Identifier: Apache-2.0
#

global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "zipkin"
    scrape_interval: 5s
    metrics_path: "/prometheus"
    static_configs:
      - targets: ["zipkin:9411"]
```

#### Docker-compose

```yaml
# This file uses the version 2 docker-compose file format, described here:
# https://docs.docker.com/compose/compose-file/#version-2
#
# This runs the zipkin and zipkin-mysql containers, using docker-compose's
# default networking to wire the containers together.
#
# Note that this file is meant for learning Zipkin, not production deployments.

version: "3.8"

services:
  storage:
    image: ghcr.io/openzipkin/zipkin-mysql:${TAG:-latest}
    container_name: mysql
    ports:
      - 3306:3306
  # The zipkin process services the UI, and also exposes a POST endpoint that
  # instrumentation can send trace data to.
  zipkin:
    image: ghcr.io/openzipkin/zipkin:${TAG:-latest}
    container_name: zipkin
    # Environment settings are defined here https://github.com/openzipkin/zipkin/blob/master/zipkin-server/README.md#environment-variables
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_HOST=storage
      # Add the baked-in username and password for the zipkin-mysql image
      - MYSQL_USER=zipkin
      - MYSQL_PASS=zipkin
    ports:
      # Port used for the Zipkin UI and HTTP Api
      - 9411:9411
    # Uncomment to enable debug logging
    # command: --logging.level.zipkin2=DEBUG
    depends_on:
      storage:
        condition: service_healthy
  dependencies:
    image: ghcr.io/openzipkin/zipkin-dependencies
    container_name: dependencies
    user: root
    entrypoint: /usr/sbin/crond -f
    environment:
      - STORAGE_TYPE=mysql
      - MYSQL_HOST=storage
      # Add the baked-in username and password for the zipkin-mysql image
      - MYSQL_USER=zipkin
      - MYSQL_PASS=zipkin
    depends_on:
      storage:
        condition: service_healthy
  prometheus:
    # Use a quay.io mirror to prevent build outages due to Docker Hub pull quotas
    # Use latest from https://quay.io/repository/prometheus/prometheus?tab=tags
    image: quay.io/prometheus/prometheus:v2.48.0
    container_name: prometheus
    ports:
      - 9090:9090
    depends_on:
      zipkin:
        condition: service_healthy
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    # Use a quay.io mirror to prevent build outages due to Docker Hub pull quotas
    # Use latest from https://quay.io/repository/app-sre/grafana?tab=tags
    image: quay.io/giantswarm/grafana:7.5.4
    container_name: grafana
    ports:
      - 3000:3000
    depends_on:
      - prometheus
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin

  setup_grafana_datasource:
    # This is an arbitrary small image that has curl installed
    # Use a quay.io mirror to prevent build outages due to Docker Hub pull quotas
    # Use latest from https://quay.io/repository/quay.io/rackspace/curl?tab=tags
    image: quay.io/cilium/alpine-curl:v1.8.0
    container_name: setup_grafana_datasource
    depends_on:
      - grafana
    volumes:
      - ./prometheus/create-datasource-and-dashboard.sh:/tmp/create.sh:ro
    working_dir: /tmp
    entrypoint: /tmp/create.sh
```
