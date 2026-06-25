# Docker Orchestra Public

Collection of standalone Docker Compose stacks for self-hosted services. Each service lives in its own directory and can be started independently.

Most stacks attach their public web container to a shared external Docker network named `router`. This lets a reverse proxy, usually `npm` / Nginx Proxy Manager, reach every service by container hostname without publishing every service port on the host.

## Prerequisites

- Docker Engine
- Docker Compose v2, available as `docker compose`
- A Linux host with permission to run Docker commands
- DNS or local host entries for the services you want to expose

## First-Time Setup

Create the shared `router` network once on the Docker host:

```sh
docker network create router
```

If the network might already exist, use this idempotent form:

```sh
docker network inspect router >/dev/null 2>&1 || docker network create router
```

This is required because every compose file declares:

```yaml
networks:
  router:
    external: True
```

Compose will not create external networks automatically. If `router` is missing, `docker compose up` will fail.

## Running a Stack

From the repository root, enter the service directory and start it:

```sh
cd npm
docker compose up -d
```

Use the same pattern for any other service:

```sh
cd <service-directory>
docker compose config
docker compose pull
docker compose up -d
docker compose logs -f
```

Stop a stack with:

```sh
docker compose down
```

Named Docker volumes are used for persistent data, so `docker compose down` does not remove application data unless you add `--volumes`.

## Environment Files and Secrets

Runtime configuration is intentionally not committed. The repository ignores:

- `.env`
- `.secrets/`
- `appdata/`

Create a `.env` file inside each service directory that needs environment variables. The required variables are the `${...}` placeholders in that service's `docker-compose.yml`.

Example:

```sh
cd bookstack
mkdir -p .secrets
printf 'bookstack-db-password' > .secrets/db_user_pwd
printf 'bookstack-root-password' > .secrets/db_root_pwd
chmod 600 .secrets/*
```

Example `.env` for a database-backed app:

```dotenv
DB_NAME=bookstack
DB_USER=bookstack
APP_URL=https://bookstack.example.com
TIME_ZONE=UTC
```

Common secret file names used by the stacks:

- `.secrets/db_user_pwd`
- `.secrets/db_root_pwd`
- `.secrets/db_url`
- `.secrets/db_user_name`

Common environment variables used across stacks include `DB_NAME`, `DB_USER`, `TZ`, `TIME_ZONE`, `LOCAL_IP`, `VPN_IP`, and service-specific URL, path, SMTP, or admin credentials.

## Reverse Proxy Setup

Start `npm` first if you want a central reverse proxy:

```sh
cd npm
docker compose up -d
```

The Nginx Proxy Manager stack publishes ports `80` and `443` on the host. Its admin UI is exposed on port `81` using `LOCAL_IP` and `VPN_IP` from `npm/.env`.

Because `npm` and the web containers are on the same `router` network, proxy hosts can target service hostnames such as:

- `bookstack`
- `owncloud`
- `paperlessngx`
- `vaultwarden`
- `wordpress`
- `ghost`
- `metabase`

Use each application's internal container port from the upstream image documentation or the service itself. Database and cache containers are normally only attached to the stack's private default network.

## Services

| Directory | Hostname | Includes |
| --- | --- | --- |
| `authelia` | `authelia` | Authelia, Postgres, Redis |
| `bookstack` | `bookstack` | BookStack, MariaDB |
| `dashy` | `dashy` | Dashy |
| `drawio` | `drawio` | draw.io |
| `excalidraw` | `excalidraw` | Excalidraw |
| `firefly-iii` | `fireflyiii` | Firefly III, MariaDB |
| `flatnotes` | `flatnotes` | Flatnotes |
| `ghost` | `ghost` | Ghost, MySQL |
| `homarr` | `homarr` | Homarr |
| `linkstack` | `linkstack` | LinkStack |
| `metabase` | `metabase` | Metabase, Postgres |
| `monica` | `monica` | Monica, MariaDB |
| `mysql` | `mysql` | Standalone MySQL |
| `npm` | `npm` | Nginx Proxy Manager, MariaDB |
| `owncloud` | `owncloud` | ownCloud, MariaDB, Redis |
| `paperless-ngx` | `paperlessngx` | Paperless-ngx, Postgres, Redis |
| `portainer` | `portainer` | Portainer CE |
| `rdt-client` | `rdtclient` | RDT Client |
| `static-web-server` | `staticwebserver` | Static Web Server |
| `stirling-pdf` | `stirlingpdf` | Stirling PDF |
| `uptime-kuma` | `uptime-kuma` | Uptime Kuma |
| `vaultwarden` | `vaultwarden` | Vaultwarden, MariaDB |
| `watchtower` | `watchtower` | Watchtower |
| `wordpress` | `wordpress` | WordPress, MariaDB |

## Updating

Update one stack at a time:

```sh
cd <service-directory>
docker compose pull
docker compose up -d
```

`watchtower` is also included if you prefer automated image update monitoring.

## Troubleshooting

If you see an error similar to `network router declared as external, but could not be found`, create the network:

```sh
docker network create router
```

If Compose warns that variables are unset, add the missing values to that service's `.env` file.

If a service starts but the reverse proxy cannot reach it, confirm both containers are attached to `router`:

```sh
docker network inspect router
```
