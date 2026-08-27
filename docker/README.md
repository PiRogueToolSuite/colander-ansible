
Useful URls:

- http://traefik.local:8080/
- https://colander.local/
- https://threatr.local/
- https://cyberchef.local/


TODO:

- [ ] we use redis 6, redis 8 is available
- [ ] redis: we could use persistence (`compose-base.yml`) :
  ```yaml
    # https://hub.docker.com/_/redis#start-with-persistent-storage
    volumes:
      - ${REDIS_VOLUME_PATH:-./data/redis}:/data 
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
