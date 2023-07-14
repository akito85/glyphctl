```
version: "3.7"

networks:
  mtaj-network:
    driver: bridge

services:
  app:
    container_name: mtaj-be
    build:
      context: ./app/mtaj-be/infra/app
      dockerfile: Dockerfile
      args:
        NGINX_CLIENT_MAX_BODY_SIZE: 10M
    image: mtaj-be
    restart: unless-stopped
    tty: true
    working_dir: /var/www
    volumes:
      - ./be/:/var/www
      - ./app/mtaj-be/infra/app/uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
    networks:
      - mtaj-network

  db:
    container_name: pgsql-mtaj
    build:
      context: ./app/mtaj-be/infra/pgsql
      dockerfile: Dockerfile
    image: pgsql-mtaj
    restart: unless-stopped
    expose:
      - "5432"
    ports:
      - "5433:5432"
    volumes:
      - ./app/mtaj-be/infra/pgsql/script:/docker-entrypoint-initdb.d
      - ./app/mtaj-be/infra/pgsql/data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=root@btpn
    networks:
      - mtaj-network

  nginx:
    image: nginx:1.19.8-alpine
    container_name: nginx-mtaj
    restart: unless-stopped
    tty: true
    ports:
      - 9876:80
    volumes:
      - ./app/mtaj-be:/var/www
      - ./app/mtaj-be/infra/nginx/conf:/etc/nginx/conf.d
    networks:
      - mtaj-network

  fe:
    container_name: mtaj-fe
    build:
      context: ./app/mtaj-fe
      dockerfile: Dockerfile
    image: mtaj-fe
    working_dir: /usr/app/src
    restart: unless-stopped
    tty: true
    volumes:
      - ./app/mtaj-fe:/usr/app/src
    ports:
      - 9875:3000
    command: npm start
    networks:
      - mtaj-network
```