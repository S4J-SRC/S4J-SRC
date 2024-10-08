# Changes include:
Config files such as
- settings.yml
- uwsgi.ini
- .env
- Caddyfile
- docker-compose.yml
- rewrite-hosts.yml
- remove-hosts.yml

And others as seen in the docker.yml
```
      - type: bind
        source: ./changes/no_results.html
        target: /usr/local/searxng/searx/templates/simple/messages/no_results.html
      - type: bind
        source: ./changes/s4j.png
        target: /usr/local/searxng/searx/static/themes/simple/img/searxng.png
      - type: bind
        source: ./changes/index.html
        target: /usr/local/searxng/searx/templates/simple/index.html
      - type: bind
        source: ./changes/base.html
        target: /usr/local/searxng/searx/templates/simple/base.html
      - type: bind
        source: ./changes/404.html
        target: /usr/local/searxng/searx/templates/simple/404.html
      - type: bind
        source: ./changes/about.md
        target: /usr/local/searxng/searx/infopage/en/about.md
      - type: bind
        source: ./changes/search-syntax.md
        target: /usr/local/searxng/searx/infopage/en/search-syntax.md
      - type: bind
        source: ./changes/about.md
        target: /usr/local/searxng/searx/infopage/id/about.md
      - type: bind
        source: ./changes/search-syntax.md
        target: /usr/local/searxng/searx/infopage/id/search-syntax.md
      - type: bind
        source: ./changes/about.md
        target: /usr/local/searxng/searx/infopage/de/about.md
      - type: bind
        source: ./changes/search-syntax.md
        target: /usr/local/searxng/searx/infopage/de/search-syntax.md
```

# searxng-docker

Create a new SearXNG  instance in five minutes using Docker

## What is included ?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [Caddy](https://github.com/caddyserver/caddy) | Reverse proxy (create a LetsEncrypt certificate automatically) | [caddy/caddy:2-alpine](https://hub.docker.com/_/caddy) | [Dockerfile](https://github.com/caddyserver/caddy-docker) |
| [SearXNG](https://github.com/searxng/searxng) | SearXNG by itself | [searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng) | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile) |
| [Redis](https://github.com/redis/redis) | In-memory database | [redis:alpine](https://hub.docker.com/_/redis) | [Dockerfile-alpine.template](https://github.com/docker-library/redis/blob/master/Dockerfile-alpine.template) |

## How to use it
- [Install docker](https://docs.docker.com/install/)
- Get searxng-docker
  ```sh
  cd /usr/local
  git clone https://github.com/searxng/searxng-docker.git
  cd searxng-docker
  ```
- Edit the [.env](https://github.com/searxng/searxng-docker/blob/master/.env) file to set the hostname and an email
- Generate the secret key `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
- Edit the [searxng/settings.yml](https://github.com/searxng/searxng-docker/blob/master/searxng/settings.yml) file according to your need
- Check everything is working: `docker compose up`
- Run SearXNG in the background: `docker compose up -d`

> [!WARNING]  
> If you use an older version of docker desktop (`< 3.6.0`), you may have to install Docker Compose v1.
> Accordingly, you should modify the commands in this documentation to suit Docker Compose v1. For instance, change 'docker compose up' to 'docker-compose up'.
>
> [Install the docker-compose plugin](https://docs.docker.com/compose/install/#scenario-two-install-the-compose-plugin) (be sure that docker-compose version is at least 1.9.0)

## How to access the logs
To access the logs from all the containers use: `docker compose logs -f`.

To access the logs of one specific container:
- Caddy: `docker compose logs -f caddy`
- SearXNG: `docker compose logs -f searxng`
- Redis: `docker compose logs -f redis`

### Start SearXNG with systemd

You can skip this step if you don't use systemd.

- ```cp searxng-docker.service.template searxng-docker.service```
- edit the content of ```WorkingDirectory``` in the ```searxng-docker.service``` file (only if the installation path is different from /usr/local/searxng-docker)
- Install the systemd unit:
  ```sh
  systemctl enable $(pwd)/searxng-docker.service
  systemctl start searxng-docker.service
  ```

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allow the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users wants to disable the image proxy, you have to modify [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:
- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
git pull
docker compose pull
docker compose up -d
```

Or the old way (with the old docker-compose version):
```sh
git pull
docker-compose pull
docker-compose up -d
```