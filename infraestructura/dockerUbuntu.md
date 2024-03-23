

# Abrir el puerto al exterior

La idea es mi red local poder lanzar los comandos de forma externa.

> Estoy siendo esta enlace https://docs.docker.com/config/daemon/remote-access/

> La segunda opción no nos ha funcionado, por lo cual he optado por la primera
## Instalacion

1. Editamos el fichero de configuración
   ```bash
   sudo systemctl edit docker.service
   ```
2. Añadimos las siguientes lineas
  ```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
  ```
> Es importante poner la linea vacia, ya que al parecer es necesario par el comando que la linea sea resete para que se pueda añadir.
3. Guardamos el fichero.
4. Reload del demonio
```bash
sudo systemctl daemon-reload
```
5. Restauramos el docker
```bash
sudo systemctl restart docker.service
```
6. Verificamos
```
sudo netstat -lntp | grep dockerd
```
## Configurar el cliente en docker en mac

1. Configuramos la variable DOCKER_HOST
 ```bash
  export DOCKER_HOST=<IP>
 ```  
2. Configuramos la variable DOCKER_PORT
```bash
  export DOCKER_PORT=2375
```

> Si esta cifrado hay que configurar DOCKER_TLS_VERIFY

## Error (Cannot connect to the Docker daemon at tcp://<IP>:2375. Is the docker daemon running?)

