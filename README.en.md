# Oricode Docker Starter

Reusable Docker infrastructure starter for PHP applications with Caddy as the edge proxy, automated HTTPS, MySQL, Redis, cron jobs, and optional Cloudflare Tunnel support. Designed for local development, staging, and self-hosted deployments.

[Documentation francaise](./README.md)

## ✨ Overview

This repository provides a ready-to-clone infrastructure foundation for web projects that need a simple, readable, and modular stack.

The goal is not to ship business logic, but to provide an execution layer that can serve as a starting point for:
- a traditional PHP application
- a multi-domain or multi-subdomain platform
- a shared local development environment
- a staging or preproduction stack
- a self-hosted setup with automated HTTPS

The project intentionally stays compact:
- one central `docker-compose.yml`
- service-specific configuration under `.docker/`
- one `.env` file to drive runtime behavior
- a Caddy-based edge layer

## 🎯 Project Goals

This starter aims to balance simplicity and adaptability.

It is designed to:
- get a stack running quickly
- remain easy to understand
- adapt to multiple deployment contexts
- avoid publishing secrets or runtime state
- provide a clean base for either public or private repositories

## 🧱 Included Components

### Caddy

`Caddy` is the HTTP/HTTPS entry point of the stack.

It handles:
- TLS termination
- HTTP to HTTPS redirects
- domain and subdomain routing
- automated certificate issuance through DNS challenge
- forwarding traffic to the PHP application

### MySQL

`MySQL` provides relational persistence.

Typical use cases:
- application data
- business tables
- primary persistent storage

### Redis

`Redis` is intended for:
- application cache
- sessions
- queues
- temporary runtime storage

### Cron

The `cron` service isolates scheduled jobs into a dedicated container.

This helps with:
- recurring task execution
- cleaner separation of responsibilities
- easier maintenance of scheduled jobs

### Cloudflare Tunnel

A tunnel template is included for setups that want to:
- expose SSH in a controlled way
- route the main domain through a tunnel
- centralize ingress without opening every port directly

## 🗺️ High-Level Architecture

```text
Internet / Tunnel
        |
        v
      Caddy
        |
        v
   PHP application
     |        |
     v        v
   MySQL    Redis
```

## 📁 Detailed Structure

```text
.
|-- .docker/
|   |-- caddy/
|   |   |-- Caddyfile
|   |   |-- Dockerfile
|   |   `-- caddyfile.json
|   |-- cloudflared/
|   |   `-- config/
|   |       `-- config.yml.template
|   |-- cron/
|   |   |-- Dockerfile
|   |   `-- task
|   |-- logs/
|   |-- mysql/
|   |   |-- Dockerfile
|   |   |-- my.cnf
|   |   `-- sql/
|   `-- nginx/
|       |-- default.conf
|       `-- oricode.com.conf
|-- .env.example
|-- .gitignore
|-- AUTHORS.md
|-- CONTRIBUTING.md
|-- LICENSE
|-- README.en.md
|-- README.md
|-- SECURITY.md
`-- docker-compose.yml
```

## 🧭 Directory Roles

- `.docker/caddy/`
  Edge, TLS, and reverse proxy configuration.
- `.docker/cloudflared/`
  Tunnel template configuration.
- `.docker/cron/`
  Cron image definition and scheduled task file.
- `.docker/logs/`
  Local log mount points.
- `.docker/mysql/`
  MySQL configuration and SQL bootstrap location.
- `.docker/nginx/`
  Reference or compatibility Nginx configurations.

## ⚡ Quick Start

### 1. Create the environment file

```powershell
Copy-Item .env.example .env
```

### 2. Fill in the variables

At minimum, review:
- MySQL credentials
- Redis settings if needed
- domain values
- Cloudflare variables if you use DNS challenge or tunnel routing

### 3. Review the entry points

Before the first launch, check:
- `docker-compose.yml`
- `.docker/caddy/Caddyfile`
- `.docker/cloudflared/config/config.yml.template`
- `.env`

### 4. Build and start

```powershell
docker compose up -d --build
```

### 5. Stop the stack

```powershell
docker compose down
```

## 🔧 Environment Variables

The [`.env.example`](./.env.example) file is the reference template.

### MySQL Block

- `DB_CONNECTION`
  Application database driver.
- `DB_HOST`
  MySQL host reachable from containers.
- `DB_PORT`
  Internal MySQL port.
- `FORWARD_DB_PORT`
  Host port exposed by Docker.
- `DB_DATABASE`
  Database created at startup.
- `DB_USERNAME`
  Application database user.
- `DB_PASSWORD`
  Application and root password in this stack variant.

### Redis Block

- `REDIS_HOST`
  Redis host.
- `REDIS_PASSWORD`
  Redis password if enabled.
- `REDIS_PORT`
  Internal Redis port.
- `FORWARD_REDIS_PORT`
  Host port exposed by Docker.
- `REDIS_CLIENT`
  Client expected by the application.

### TLS / Cloudflare Block

- `CLOUDFLARE_API_TOKEN`
  Token used for DNS challenge.
- `CLOUDFLARE_API_MAIL_PRIMARY`
  Primary email used for TLS automation.
- `CLOUDFLARE_API_MAIL`
  Secondary email consumed by some parts of the stack.
- `FORWARD_CADDY_PORT`
  Host HTTP port.
- `FORWARD_CADDY_SSL_PORT`
  Host HTTPS port.

### Runtime / Mapping Block

- `DOCKER_DB_CONTAINER_NAME`
  MySQL container name.
- `WWWGROUP`
  GID inside containers.
- `WWWUSER`
  UID inside containers.
- `WWWWORKDIR`
  Working path mounted into the web runtime.

### Domain / Tunnel Block

- `TUNNEL_UUID`
  Tunnel identifier.
- `DOMAIN`
  Primary domain.
- `HOST_HOSTNAME`
  Hostname prefix for technical endpoints.

## 🌐 Reverse Proxy and TLS

The main files are:
- [`.docker/caddy/Caddyfile`](./.docker/caddy/Caddyfile)
- [`.docker/caddy/caddyfile.json`](./.docker/caddy/caddyfile.json)

The `Caddyfile` is the easiest source to maintain.

The JSON file can still be useful if you want to:
- compare exported configuration
- keep a machine-readable representation
- prepare config generation workflows later

## ☁️ Remote Tunnel

The [`.docker/cloudflared/config/config.yml.template`](./.docker/cloudflared/config/config.yml.template) file can be adapted to your environment.

Typical use cases:
- exposing SSH through a controlled hostname
- routing the main domain to the reverse proxy
- routing wildcard subdomains through the same edge layer

## 🗃️ Database and Cache

### MySQL

Related files:
- [`.docker/mysql/Dockerfile`](./.docker/mysql/Dockerfile)
- [`.docker/mysql/my.cnf`](./.docker/mysql/my.cnf)

The `.docker/mysql/sql/` directory can be used for:
- initialization scripts
- seed data
- bootstrap SQL patches

### Redis

Redis intentionally stays minimal so it can act as a lightweight support service that is easy to replace or extend later.

## ⏰ Scheduled Jobs

The cron service relies on:
- [`.docker/cron/Dockerfile`](./.docker/cron/Dockerfile)
- [`.docker/cron/task`](./.docker/cron/task)

Typical use cases:
- recurring jobs
- maintenance routines
- internal sync tasks
- cron-driven queue processing

## 🛡️ Ignored Files and Publication Safety

The repository is prepared to stay shareable without bundling sensitive local state.

The `.gitignore` covers:
- `.env`
- `.idea/`
- certificates and ACME data
- Caddy runtime state
- logs
- local MySQL data
- local Redis data

## 🔒 Security Best Practices

- never commit a real `.env`
- never commit generated certificates
- never commit private keys
- keep Cloudflare token permissions narrow
- rotate secrets if exposure is suspected
- inspect Git history before making a repository public

For the disclosure process, see [SECURITY.md](./SECURITY.md).

## 🧩 Possible Use Cases

This starter can fit:
- a single-domain project
- a subdomain-based multi-tenant SaaS
- a demo or staging stack
- a shared team development environment
- a reusable infrastructure template

## 🛠️ Fast Customization Points

You can quickly customize:
- domains
- subdomains
- Docker images
- exposed ports
- mounted volumes
- TLS strategy
- Caddy hosts and routes
- cron jobs

## 🧪 Recommended Manual Checks

Before pushing publicly:
- verify `.env` is not tracked
- verify certificates are not tracked
- verify runtime files are not tracked
- verify exposed domains are intentional
- verify mounted paths are generic enough
- verify Git history does not contain old secrets

## 🆘 Troubleshooting

### Port 80 or 443 is already in use

Adjust:
- `FORWARD_CADDY_PORT`
- `FORWARD_CADDY_SSL_PORT`

### MySQL or Redis port is already used

Adjust:
- `FORWARD_DB_PORT`
- `FORWARD_REDIS_PORT`

### Certificate issuance fails

Check:
- Cloudflare token
- token permissions
- DNS zone
- configured domain in `.env`
- Caddy configuration

### Tunnel routing does not work

Check:
- `TUNNEL_UUID`
- `DOMAIN`
- `HOST_HOSTNAME`
- cloudflared template
- DNS resolution

## 🤝 Contributing

Contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## 📜 License

This project is released under the MIT License. See [LICENSE](./LICENSE).

## 👤 Maintainer

- Website: <https://www.adrielzimbril.com/>
- GitHub: <https://github.com/adrielzimbril>

## 📝 Conclusion

This repository is an infrastructure foundation. It does not try to hide complexity behind too much tooling; instead, it provides a clear, publishable, and reusable base for modern web projects.
