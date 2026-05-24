TODO:

- [ ] we use redis 6, redis 8 is available
- [ ] redis: we could use persistence (`compose-base.yml`) :
  ```yaml
    # https://hub.docker.com/_/redis#start-with-persistent-storage
    volumes:
      - ${REDIS_VOLUME_PATH:-./data/redis}:/data 
    command: "redis-server --save 60 1"
  ```
