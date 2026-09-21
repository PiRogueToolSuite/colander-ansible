# Using docker to deploy the PTS stack.

This folder contains the docker compose definition files which allows you to deploy the whole PTS stack or pick
only the component(s) you want.

## Starting the whole stack

The following command should get you up and running fairly quickly (by default, you may be able to start straight away.
But in some case you may need to adjust the `.env` file, so take a look to it. It should be self-explanatory):

```bash
docker compose -f compose-dev.yml up -d
```

This will :

- create a self signed certificate (allowing you to use pirogue tooling to connect to colander without requiring a certificate
  issued by a Certificate Authority).
- start the whole stack (colander, threatr, mandolin, etc.)
  - you'll get access to:
    - [Colander](https://colander.local/)
    - [Threatr](https://threatr.local/)
    - [Cyberchef](https://cyberchef.local/)
    - [Traefik dashboard](http://traefik.local:8080/)

Then you can create admin users on both colander and threatr:

```bash
docker compose -f compose-dev.yml exec colander-front  /entrypoint python manage.py createsuperuser
docker compose -f compose-dev.yml exec threatr-front  /entrypoint python manage.py createsuperuser
```

Then connect add a threatr integration (As a reminder this is documented [here](https://pts-project.org/docs/colander/deployment/)):

- Go to https://threatr.local/admin ; create a user 'colander-user' and its token
- Go to https://colander.local/admin ; then add a backend credential 'threatr' with a value of:
  `{ "api_key": "thetoken"}`

**note:** You can also run `docker compose up -d` but you'll not get access to Traefik's dashboard nor generate a self signed certificate.
`compose.yml` is the base file can be used as a base for production deployments. `compose-dev.yml` overrides some of the service definitions
of the `compose.yml`.


## Starting the components you need (self-service/standalone mode)

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

### Compose your stack "à la carte"

Of course, you can pick more than one tool by giving the corresponding compose file as a value of `-f` argument of
the docker compose CLI:

```bash
docker compose -f compose-colander.yml -f compose-threatr.yml up -d
```

---


Useful URls:

TODO:

- [x] publish threater-postgres & colander-postgres
  - https://github.com/PiRogueToolSuite/postgres/pkgs/container/postgres
- [x] publish new images for colander/threatr (to add `EXPOSE 5000`)
  - [x] remove `expose: 5000` from `compose-colander.yml` `compose-threatr.yml` files
- [x] check if volumes are in the same between now and what was done in production (eg for postgres backups and data for threatr and colandr)
- [x] redis: we could use persistence (`compose-base.yml`) :
  ```yaml
    # https://hub.docker.com/_/redis#start-with-persistent-storage
    volumes:
      - ${REDIS_VOLUME_PATH:-production_redis_data}:/data 
    command: "redis-server --save 60 1"
  ```
- [x] we use redis 6, redis 8 is available

- [ ] hardening: readonly/tmpfs containers, cap_drop ALL, security_opt no-new-privileges security_opts ; .... https://docs.docker.com/compose/trust-model/
- [ ] dans `roles/colander/tasks/configure.yml`, les dockerfile postgres et traefik ont été déplacés vers docker/compose/{postgres,traefik}/Dockerfile.
      il faut s'assurer qu'on a pas besoin de surcharger ça avec ansible ; ou du moins gérer le build des images basé
      sur un env ou équivalent :
      `./templates/traefik/Dockerfile.j2`
      `./templates/postgres/Dockerfile.j2`
- [ ] # FIXME implement: stack.services.traefik.vars.enable_dashboard
