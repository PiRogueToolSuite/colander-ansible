ARG POSTGRES_IMAGE_VERSION=latest
FROM postgres:$POSTGRES_IMAGE_VERSION

COPY ./compose/postgres/maintenance /usr/local/bin/maintenance
RUN chmod +x /usr/local/bin/maintenance/*
RUN mv /usr/local/bin/maintenance/* /usr/local/bin \
    && rmdir /usr/local/bin/maintenance
