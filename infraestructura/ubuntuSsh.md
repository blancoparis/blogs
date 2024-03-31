# Ubuntu

## Instalamos

> Enlace (https://www.ionos.es/digitalguide/servidores/configuracion/ubuntu-ssh/)

```bash
sudo apt install openssh-server
```

## Activamos el servicio

```bash
sudo systemctl status ssh
```

## Acviar el puerto

```bash
sudo ufw allow ssh
``
## Configurar el ssh

```bash
sudo nano /etc/ssh/sshd_config
``
pdb-GF75-Thin-9SD
