# shake

host your own web-based quake lan party.

## running

clone this repo and run `docker compose up -d`.

visit `https://<server-ip>` and play.

to keep things simple every visitor will join the same lobby by default, but if you want a private game you can append `?server=whatever` and share that link

don't have friends? no problem, activate bots by appending `?lonely`

want to play a different map from the demo? append `?map=q3dm7` (options are `q3dm1`, `q3dm7`, `q3dm17`, `q3tourney2`)

## local HTTPS setup (Caddy reverse proxy)

the stack uses [Caddy](https://caddyserver.com/) as a reverse proxy to provide HTTPS on your local network using a self-signed certificate (`tls internal`). Let's Encrypt cannot issue certificates for raw IP addresses, so Caddy generates a local root CA instead.

### steps

1. find your local machine IP:
   ```bash
   hostname -I | awk '{print $1}'
   ```

2. edit `Caddyfile` and replace `<your-ip>` with your IP.

3. start the stack:
   ```bash
   docker compose up -d
   ```

4. trust the local CA (one-time per device/browser):
   ```bash
   docker exec caddy caddy trust
   ```
   or copy the root certificate from the `caddy_data` volume at
   `caddy/pki/authorities/local/root.crt` and import it into your OS / browser trust store.

### routes

| URL | Service |
|-----|---------|
| `https://<your-ip>` | shake client (nginx) |
| `https://<your-ip>:8443` | shake peer server |

### verifying routing

check that all containers are running:
```bash
docker compose ps
```

confirm the nginx config inside `shake-dev` is valid:
```bash
docker exec shake-dev nginx -t
```

test that Caddy can reach the nginx container on the internal network:
```bash
docker exec caddy curl -I http://shake-dev:80
```

### nginx entrypoint log notes

when `shake-dev` starts you will see lines like:

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
Configuration complete; ready for start up
```

these are **normal** — the nginx image runs a startup script that detects your custom `nginx.conf` is different from its default and then starts nginx successfully. only worry if you see lines containing `[emerg]` (config parse errors) or repeated container restarts.

## disclaimer

this is a very minimal scrape and remix of the extremely cool https://thelongestyard.link/ whose source can be found [here](https://github.com/jdarpinian/ioq3) and [here](https://github.com/jdarpinian/HumbleNet)

inspired by how i've previously used the now-abandoned [quake-kube](https://github.com/criticalstack/quake-kube)