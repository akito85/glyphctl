
```
docker network create --subnet=172.20.0.0/16 customnetwork
```
```
docker run --net customnetwork --ip 172.20.0.10 -d container
```
```
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' name_or_id
```
```version: '2'
services:
  webserver:
    image: nginx
    container_name: web-server
    networks:
      customnetwork:
        ipv4_address: 172.20.0.10
networks:
  customnetwork:
    ipam:
      config:
        - subnet: 172.20.0.0/16
