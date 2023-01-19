# Traefik - Portainer Docker Template

This is a docker template for Traefik and Portainer. Traefik being used as a reverse proxy and Portainer as a container management tool. Traefik is configured to use cloudflare for DNS services and Let's Encrypt for SSL certificates.

![Visitor](https://visitor-badge.laobi.icu/badge?page_id=manh21.traefik-portainer)

## Usage

Depending on what type of service or setup your are using, you can use the following templates to get started.

### Config files with same docker network

System that belongs to the same docker network can be accessed by using label as seen on the example below.

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.portainer.entrypoints=http"
    - "traefik.http.routers.portainer.rule=Host(`portainer.local.example.com`)"
    - "traefik.http.middlewares.portainer-https-redirect.redirectscheme.scheme=https"
    - "traefik.http.routers.portainer.middlewares=portainer-https-redirect"
    - "traefik.http.routers.portainer-secure.entrypoints=https"
    - "traefik.http.routers.portainer-secure.rule=Host(`portainer.local.example.com`)"
    - "traefik.http.routers.portainer-secure.tls=true"
    - "traefik.http.routers.portainer-secure.service=portainer"
    - "traefik.http.services.portainer.loadbalancer.server.port=9000"
    - "traefik.docker.network=proxy"
```

The only thing that needs to be changed is the host name, the service name and the port number.

- Host name: ```- "traefik.http.routers.portainer.rule=Host(`portainer.local.example.com`)"``` and ```- "traefik.http.routers.portainer-secure.rule=Host(`portainer.local.example.com`)"```
- Service port: ```- "traefik.http.services.portainer.loadbalancer.server.port=9000"```
- Service Name: ```- "traefik.http.routers.portainer-secure.service=portainer"```

### Config files for service with different server / network / docker host

System that belongs to a different docker network or a different server can be accessed by using the following template. This file can be located at `./traefik/data/config.yml`.

NOTE: Everytime you make changes to this file, you need to restart traefik with `docker-compose up -d --force-recreate` to apply the changes.

```yaml
#region routers 
  routers:
    proxmox:
      entryPoints:
        - "https"
      rule: "Host(`proxmox.local.example.com`)"
      middlewares:
        - default-headers
      tls: {}
      service: proxmox
#region services
  services:
    proxmox:
      loadBalancer:
        servers:
          - url: "https://192.168.0.100:8006"
        passHostHeader: true
```

Main thing that's will work with this template is the `routers` and `services` section. The `routers` section is where you define the host name and the service target name on services region. The `services` section is where you define the origin of server url and the port number.

- Host name: ```- "Host(`proxmox.local.example.com`)"``` this is the entry point for your application.
- Service: ```proxmox``` this is the service name that you will use must be valid name inside services region.
- Server URL: ```- url: "https://192.168.0.100:8006"``` this is your service origin endpoint.

## Installation

- Create New Network for Traefik
  - ```bash docker network create proxy```

- Spin Up Traefik
  - ```bash cd  ./portainer```

  - ```bash docker-compose up -d```

- Spin Up Portainer

  - ```bash cd ./traefik```

  - ```bash docker-compose up -d```

- Update config file for traefik external docker/portainer services

  - ```bash /traefik/data/config.yml```

  - Start traefik with ```bash docker-compose up -d --force-recreate```

## Miscalenous

### Generate Basic Auth Password

- ```bash sudo apt update```

- ```bash sudo apt install apache2-utils```

- ```bash echo $(htpasswd -nb "<USER>" "<PASSWORD>") | sed -e s/\\$/\\$\\$/g```
NOTE: Replace <USER> with your username and <PASSWORD> with your password to be hashed.

- Paste the output in your docker-compose.yml in line (traefik.http.middlewares.traefik-auth.basicauth.users=<USER>:<HASHED-PASSWORD>)

## Credit

Thanks for techno tim for the awesome tutorial on how to setup traefik and portainer. [Video](https://www.youtube.com/watch?v=liV3c9m_OX8)
