# Remote CTRL

Remote CTRL is a self-hosted, browser-based console for your servers: SSH
terminals, SFTP file transfers, and RDP and VNC desktops, all from one tab. It
runs as a single Docker image.

This repository holds the installation files and license. Release notes are
in [Releases](../../releases). Report bugs and request features in
[Issues](../../issues).

## Features

- SSH terminals with command suggestions, snippets and saved password insertion
- SFTP browser with resumable uploads of any size
- Embedded RDP and VNC sessions through Apache Guacamole
- Jump hosts, folders, favorites and team sharing
- AI agents through MCP, scoped to the folders you choose (Enterprise)

## Editions

| | Free | Enterprise |
| --- | --- | --- |
| Users | 1 | 10, 50, 250 or more |
| SSH, SFTP, RDP, VNC | Yes | Yes |
| Teams and shared folders | Yes | Yes |
| AI agents (MCP) | No | Yes |
| Price | Free | Annual subscription |

Every account counts toward the user limit. Enterprise is activated from
**About** with a license key; each key works on one installation at a time.

## Install

Requirements: Docker with Compose, on Linux x86-64.

```sh
curl -fsSLO https://raw.githubusercontent.com/Umberto-Casaburi/remote-ctrl/main/docker-compose.yml
curl -fsSL https://raw.githubusercontent.com/Umberto-Casaburi/remote-ctrl/main/.env.example -o .env
# Replace each CHANGE_ME with a random secret:
for key in APP_SECRET RDP_GUACAMOLE_PASSWORD GUACAMOLE_POSTGRES_PASSWORD; do
  sed -i "s/^$key=CHANGE_ME$/$key=$(openssl rand -hex 32)/" .env
done
docker compose up -d
```

Open http://localhost:4000 and create the administrator account.

Data lives in the `remote-ctrl-data` volume: the app database and the
Guacamole PostgreSQL data. Back it up together with your `.env`: without the
same `APP_SECRET`, saved host passwords and keys cannot be decrypted.

To use the built-in VPN (Enterprise), uncomment the VPN lines in
`docker-compose.yml` and open UDP port 51820 on the server.

## Update

```sh
docker compose pull
docker compose up -d
```

## Reverse proxy

Forward WebSocket upgrades and the forwarded headers. Example for Nginx on a
subpath:

```nginx
location ^~ /remote/ {
  proxy_pass http://127.0.0.1:4000/;
  proxy_http_version 1.1;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-Host $host;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_set_header X-Forwarded-Prefix /remote;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  client_max_body_size 32m;
  proxy_request_buffering off;
}
```

Then set `RDP_GUACAMOLE_PUBLIC_URL=/remote/guacamole` and
`TRUST_PROXY_HOPS=1` in `.env`.

## License

Remote CTRL is proprietary software. See [LICENSE](LICENSE). It includes
third-party components under their own licenses, listed in [NOTICE](NOTICE).
