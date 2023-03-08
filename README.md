# omada-docker-portainer
Instalacion de omada controller en docker con portainer y docker compose


## Instalacion portainer + docker + docker compose + omada en almalinux 8

## Instalacion de Docker 
```
sudo su
```
```
dnf makecache --refresh
```

```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
```

```
sudo timedatectl set-timezone America/Mexico_City
```

```
dnf install -y yum-utils device-mapper-persistent-data
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
dnf install docker-ce -y
```

```
systemctl start docker && systemctl enable docker
```
## Instalacion de Docker-compose

```
curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o docker-compose
```

```
sudo mv docker-compose /usr/local/bin && sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## Omada controller desde docker con docker compose

```
nano docker-compose.yml
```

- Dentro del archivo colocamos

```
version: '3.3'
services:
  omada-controller:
    container_name: omada-controller
    image: 'mbentley/omada-controller:5.9'
    restart: unless-stopped
    network_mode: bridge
    ports:
      - '8088:8088'
      - '8043:8043'
      - '8843:8843'
      - '29810:29810/udp'
      - '29811:29811'
      - '29812:29812'
      - '29813:29813'
      - '29814:29814'
    environment:
      - MANAGE_HTTP_PORT=8088
      - MANAGE_HTTPS_PORT=8043
      - PORTAL_HTTP_PORT=8088
      - PORTAL_HTTPS_PORT=8843
      - SHOW_SERVER_LOGS=true
      - SHOW_MONGODB_LOGS=false
      - SSL_CERT_NAME=tls.crt
      - SSL_KEY_NAME=tls.key
      - TZ=America/Mexico_City
      - TLS_1_11_ENABLED=true
    volumes:
      - './omada-data:/opt/tplink/EAPController/data'
      - './omada-work:/opt/tplink/EAPController/work'
      - './omada-logs:/opt/tplink/EAPController/logs'
      - './omada-cert:/cert'
```

```
docker-compose up -d
```

- Ingresamos a la web con la https://IP:8043

## Instalacion de portainer
Lo utilizamos para administracion de los contenedores.
```
docker pull portainer/portainer
```

```
docker run -d -p 9000:9000 --name miportainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /opt/portainer:/data portainer/portainer
```

```
docker ps
sudo docker restart miportainer
```
- Ingresamos a portainer con IP:9000
- Cambia la Ip a publica
cambiar ip en Enviroments>local>Public IP
- Si algo va mal
```
dnf install firewalld -y
systemctl enable --now firewalld
firewall-cmd --permanent --add-port=8088/tcp
firewall-cmd --permanent --add-port=8043/tcp


#firewall-cmd --permanent --zone=trusted --change-interface=docker0
#firewall-cmd --permanent --zone=trusted --add-port=4243/tcp
#firewall-cmd --reload
```
