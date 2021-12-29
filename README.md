# Traefik Reverse Proxy for Local Development

## Set Up

- Using [mkcert](https://github.com/FiloSottile/mkcert) generate local certificate files

  - `mkcert -install`
  - MacOS/Linux: `mkcert --cert-file certs/cert.pem --key-file certs/key.pem "*.docker.localhost" "*.traefik.localhost"`
  - Windows: `mkcert --cert-file .\certs\cert.pem --key-file .\certs\key.pem "*.docker.localhost" "*.traefik.localhost"`

- Start the traefik container
  - `docker compose up -d --build`

## Adding containers to the proxy

### Attaching to a shared network

Attach the container providing services to the `traefik` network created by this project

```yaml
# docker-compose.yml
services:
  app:
    # other configurations
    network:
      # other networks
      - traefik
# more configurations...
networks:
  traefik:
    name: traefik
    # -or-
    external: true
```

When using the `external` option it may be better to create the network in advance `docker network create traefik` since compose projects will refuse to start if an external network is unavailable

One can also attach a single container to a network using `docker network connect traefik [container]`

### Routing traffic to your project

Add routing rules to your container using docker labels

```yaml
# docker-compose.yml
services:
  # Update this to the name of the service you want to work with in your docker-compose.yml file
  app:
    # other configurations
    labels:
      - traefik.http.routers.he.rule=Host(`something.traefik.localhost`, `something.docker.localhost`, `he.localhost`)
      - traefik.http.services.he-app.loadbalancer.server.port=80
```

-or-

`` docker run ...lots of options here -l traefik.http.routers.he.rule=Host(`something.traefik.localhost`, `something.docker.localhost`, `he.localhost`)` -l traefik.http.services.he-app.loadbalancer.server.port=80 ``
