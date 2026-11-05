# LampasPinax 1.0.94

Current public release: **1.0.94** (released 2026-10-30).

Download the player installers and packages from the [Releases](../../releases) page.

Server image:

```
docker pull ghcr.io/jawheelr/lampaspinax-server:1.0.94
```

## Running the server

Ports (the server listens inside the container):

| Port | Used for | When |
|------|----------|------|
| **8080** | Web interface and API (HTTP), the sign players and the admin | Always |
| 443 / 80 | Direct HTTPS with the built-in Caddy (certificates, ACME challenge on 80) | Only if you turn on Settings > Connections > direct HTTPS |
| 8443 / 8444 | Device channel with client certificates (8443) and its pairing/bootstrap endpoint (8444) | Only when `DEVICE_CHANNEL_MODE=mtls` |

Normally put a reverse proxy in front of 8080 and publish only 443 from the proxy. Signs and controllers are reached
outbound from the server (for example ThinkSign LED controllers on port 10001), so nothing needs to be opened inbound
for them.

Volume (everything the server keeps lives in one folder):

| Container path | What it holds |
|----------------|---------------|
| **/data** | Database, uploaded media, rendered files, the certificate authority and device certificates, logs, installed updates, and Caddy certificates |

Map `/data` to a folder on the host and back it up. If you lose it you lose your content and the certificates your
devices trust. Give the container `--shm-size 1gb` (it converts animated graphics with Chromium).

`docker-compose.yml`:

```yaml
services:
  lampaspinax:
    image: ghcr.io/jawheelr/lampaspinax-server:1.0.94
    container_name: lampaspinax
    restart: unless-stopped
    network_mode: host        # lets the server find and reach signs on your LAN; uses port 8080 on the host
    shm_size: "1gb"
    volumes:
      - ./data:/data
    environment:
      DATA_DIR: /data
      SESSION_SECRET: "put-a-long-random-value-here"
      CLIENT_ENROLL_TOKEN: "put-another-long-random-value-here"
```

Create the two secrets with `openssl rand -hex 32`, then `docker compose up -d` and open `http://<host>:8080`.

Without host networking, publish the ports instead (automatic discovery of signs on the LAN will not work):

```
docker run -d --name lampaspinax --restart unless-stopped --shm-size 1gb \
  -p 8080:8080 -v "$PWD/data:/data" \
  -e SESSION_SECRET=... -e CLIENT_ENROLL_TOKEN=... ghcr.io/jawheelr/lampaspinax-server:1.0.94
```

Add `-p 8443:8443 -p 8444:8444 -e DEVICE_CHANNEL_MODE=mtls` only for the client-certificate device channel.


This repository is a read-only distribution mirror: it contains release information (`VERSION`, `RELEASE.json`) and
release downloads only, not source code. Contributions and forks are not accepted; see [LICENSE](LICENSE).
