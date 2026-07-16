# portmgr

Port registry for running many Docker Compose projects on one box without collisions. Born from a homelab that hosts 100+ projects on a single server.

```
$ portmgr alloc shop web api db
Allocated 3 port(s) for 'shop'.

=== shop ===
Hostname:  shop.local.test

.env vars:
  SHOP_WEB_PORT=10000
  SHOP_API_PORT=10001
  SHOP_DB_PORT=10002

Written: .env.shop
Written: docker-compose.shop.yml
Caddy restarted.
```

## What it does

- **Allocates ports** from a fixed range (10000-19999 by default), tracked in `~/.portmgr/registry.json`. No two projects ever collide; re-running `alloc` is idempotent and can add services to an existing project.
- **Writes project files**: `.env.<project>` with `<PROJECT>_<SVC>_PORT` vars and a `docker-compose.<project>.yml` stub wired to them.
- **Generates reverse-proxy vhosts**: one Caddy snippet per project (`<project>.<domain>` → web port; `/api*` routes to the api port when both `web` and `api` services exist), then reloads Caddy. Skipped gracefully if Caddy isn't installed.
- **Prints /etc/hosts lines** for your laptop/other LAN machines: `portmgr hosts`.

## Commands

```
portmgr init [--domain <domain>] [--ip <ip>]   initialize registry
portmgr alloc <project> <svc> [<svc>...]       allocate (or extend) a project
portmgr free <project>                         release ports
portmgr list / show <project>                  inspect
portmgr set-hostname <project> <host>          custom vhost
portmgr hosts                                  /etc/hosts export
portmgr nginx-conf                             regenerate vhosts + reload Caddy
```

## Configuration

Everything lives in `~/.portmgr/registry.json`. Optional keys you can set by hand:

| Key | Purpose |
|-----|---------|
| `domain` | base domain for generated hostnames (default `local.test`) |
| `bind_ip` | IP printed by `portmgr hosts` (LAN or VPN IP of the server) |
| `tls_cert` / `tls_key` | if set, generated vhosts serve TLS with these files (works well with mkcert or a private CA) |
| `post_alloc_hook` | script to run after each `alloc`, receives the project name: useful for any per-project bootstrap you want automated |

## Requirements

Python 3.8+, no dependencies. Caddy optional (vhost generation), sudo for writing to `/etc/caddy/conf.d` and reloading.

## License

MIT
