
Useful URls:

- http://traefik.local:8080/
- https://colander.local/
- https://threatr.local/
- https://cyberchef.local/


TODO:

- [ ] hardening: readonly/tmpfs containers, cap_drop ALL, security_opt no-new-privileges security_opts ; .... https://docs.docker.com/compose/trust-model/
- [ ] check if volumes are in the same between now and what was done in production (eg for postgres backups and data for threatr and colandr)
- [ ] we use redis 6, redis 8 is available
- [ ] redis: we could use persistence (`compose-base.yml`) :
  ```yaml
    # https://hub.docker.com/_/redis#start-with-persistent-storage
    volumes:
      - ${REDIS_VOLUME_PATH:-production_redis_data}:/data 
    command: "redis-server --save 60 1"
  ```
- [ ] dans `roles/colander/tasks/configure.yml`, les dockerfile postgres et traefik ont été déplacés vers docker/compose/{postgres,traefik}/Dockerfile.
      il faut s'assurer qu'on a pas besoin de surcharger ça avec ansible ; ou du moins gérer le build des images basé
      sur un env ou équivalent :
      `./templates/traefik/Dockerfile.j2`
      `./templates/postgres/Dockerfile.j2`
- [ ] dans la conf traefik ; c'est compliqué de setter :
      ```yaml
      tls:
        certResolver: letsencrypt
      ```
      plusieurs pistes :
      - voir pour injecter dans le yaml quand on déploie en prod...
      - voir les dynamic config de traefik ; mais ça gère pas la composition
      - voir si on peut utiliser du conditionnel via les templates go/sprig
        https://pkg.go.dev/text/template#pkg-index
        https://masterminds.github.io/sprig/

## Migration from previous deployment method (Ansible)

- Volume definition is now in a dedicated file: `compose-volumes.yml`, check if it
  matches the configuration you have, if not create another compose file and call
  `docker compose -f compose.yml -f mycompose.yml up -d` to adapt your setup.
- For local deployments, we use a self signed certificate to allow pirogue tools to
  connect to colander without going clear text nor needing a valid certificate.

## Start PTS

### Start the whole stack (colander + threatr + mandolin + cyberchef)

```bash
# docker compose uses compose.yml by default
docker compose up -d
```

### Start the whole stack, with dev tools

- traefik: it enables [traefik API](http://traefik.local:8080/api/) / [dashboard](http://traefik.local:8080/dashboard/)
- traefik: sets log level to TRACE
- traefik: use of a self signed certificate manually generated, to allow local pirogue to connect to colander instance without
  needing a valid certificate (local dev env use only)

```shell
# generate a self-signed certificate
openssl req -new -x509 -subj "/C=FR/L=Paris/O=PTS-dev/CN=colander.local" -addext "subjectAltName=DNS:colander.local,DNS:threatr.local,DNS:cyberchef.local" -newkey rsa:2048 -keyout data/traefik/certs/key.pem -out data/traefik/certs/cert.pem -days 365 -nodes
docker compose -f compose-dev.yml up -d
```

### Start colander only

```bash
docker compose -f compose-colander.yml up -d
```

### Start threatr only

```bash
docker compose -f compose-threatr.yml up -d
```

### Start mandolin only

```bash
docker compose -f compose-mandolin.yml up -d
```

### Start cyberchef only

```bash
docker compose -f compose-cyberchef.yml up -d
```
