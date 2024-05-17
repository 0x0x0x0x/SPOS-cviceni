# Docker

## Instalace

* https://docs.docker.com/engine/install/debian/
* https://docs.docker.com/compose/install/

## Priklady

```bash
docker run hello-world
docker run --rm docker/whalesay cowsay "It works"
docker run -it --rm  --name debian debian:buster-slim /bin/bash
```

```bash
docker exec -it debian /bin/bash
```

```bash
docker ps -a
docker kill debian
docker start debian
docker inspect debian
docker rm debian
```

```bash
docker run --name apache2 -v "$PWD":/usr/local/apache2/htdocs/ -p 8080:80 -d httpd:2-alpine
```

## Build image

```bash
docker login -u user registry.repo.spos
docker build -t path/to/image:tag -f Dockerfile .
docker tag path/to/image:tag path/to/image:newtag
docker push path/to/image:newtag
```

**Dockerfile**

```dockerfile
FROM debian:11

RUN apt update && apt install -y nginx

COPY ./index.html /var/www/html/

RUN echo "Ahoj svete"

ENTRYPOINT ["/usr/sbin/nginx"]

```

**index.html**

```bash
DOCKER
```

```bash
docker build -t nginx:my -f Dockerfile .
docker images
docker tag nginx:my nginx:spos
docker run --rm -p 8181:80 nginx:my
curl 127.0.0.1:8181
# mapuje pwd na /var/www/html
docker run --name mynginx --rm -v $PWD:/var/www/html -p 8181:80 nginx:my
```

# Triky


```bash
docker system prune
docker ps -a | awk '{print $1}' | xargs docker kill 
docker ps -a | awk '{print $1}' | xargs docker rm
```

### Compose

```bash
nano docker-compose.yaml
docker compose up -d
docker compose ps
psql postgresql://postgres:heslo@127.0.0.1:15432/template1
docker compose down
```

```
services:
  nginx:
    image: nginx:latest
    ports:
      - 8181:80
    volumes:
      - ./:/usr/share/nginx/html/
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: heslo
    volumes:
      - ./dtata:/var/lib/postgresql
    ports:
      - 15432:5432
  redis:
    image: redis:latest
```
