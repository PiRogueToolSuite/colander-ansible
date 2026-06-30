ARG TRAEFIK_IMAGE_VERSION=latest
FROM traefik:$TRAEFIK_IMAGE_VERSION
RUN mkdir -p /etc/traefik/acme \
  && touch /etc/traefik/acme/acme.json \
  && chmod 600 /etc/traefik/acme/acme.json
# COPY ./compose/traefik/traefik.yml /etc/traefik
