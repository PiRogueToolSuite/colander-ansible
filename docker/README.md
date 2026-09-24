<div style="display: flex; align-items: center; flex-direction: column;">
  <img src="./android-chrome-512x512.png" alt="PTS logo" style="max-width: 100px;">
</div>

# Colander quick deploy

Welcome to Colander quick deploy.

Here you'll find all the necessary resources to get the entire Colander stack up and running as quickly as possible (or just the components you're interested in).

## Requirements

You'll need [docker and docker compose](https://docs.docker.com/compose/install/).

## Download

You can download colander quick deploy [from here](./colander-quick-deploy.zip).

This is licensed under GPLv3. You can find the sources on [our Github repository](https://github.com/PiRogueToolSuite/colander-ansible).

## Using docker to deploy the PTS stack.

The provided archive is based on [colander-ansible/docker]() folder. It contains the definition of the
PTS' architecture using docker compose.

You can start some of the tools (using `docker compose -f compose-xxx.yml up -d` command like described
below) or you can start the whole stack by just going with `docker compose up -d`

### Starting the components you need (self-service/standalone mode)

The following tools should work with their default configuration. If you want to customize your deployment, you can copy `.env.example` to `.env` and
edit the settings exposed there to match your requirements.

#### Before starting

You first need a basic .env file with django secret key for colander/threatr and minio access/secret keys:

```bash
docker compose -f compose-init.yml up -d
```

#### Start colander only

```bash
docker compose -f compose-colander.yml up -d
```

By default, colander should be available at:
- [http://localhost:5000](http://localhost:5000)
- [https://colander.local:4443](https://colander.local:4443) (if you put `colander.local` as additional hostname on line starting by `127.0.0.1`)

#### Start threatr only

```bash
docker compose -f compose-threatr.yml up -d
```

By default, threatr should be available at:
- [http://localhost:5001](http://localhost:5001)
- [https://threatr.local:4443](https://threatr.local:4443) (if you put `threatr.local` as additional hostname on line starting by `127.0.0.1`)

#### Start mandolin only

```bash
docker compose -f compose-mandolin.yml up -d
```

By default, mandolin should be available at:
- [http://localhost:5002](http://localhost:5002)
- [https://mandolin.local:4443](https://mandolin.local:4443) (if you put `mandolin.local` as additional hostname on line starting by `127.0.0.1`)

#### Start cyberchef only

```bash
docker compose -f compose-cyberchef.yml up -d
```

By default, mandolin should be available at:
- [http://localhost:5003](http://localhost:5003)
- [https://cyberchef.local:4443](https://cyberchef.local:4443) (if you put `cyberchef.local` as additional hostname on line starting by `127.0.0.1`)

### Starting a "à la carte" stack (multiple tools)

Of course, you can pick more than one tool by giving the corresponding compose file as a value of `-f` argument of
the docker compose CLI:

```bash
docker compose -f compose-colander.yml -f compose-threatr.yml up -d
```

### Starting the whole stack

The following command should get you up and running fairly quickly (by default, you may be able to start straight away.
But in some case you may need to adjust the `.env` file, so take a look to it. It should be self-explanatory):

```bash
docker compose up -d

# or the following if you cannot obtain a valid letsencrypt certificate because of your context/requirements (it
# will generate a self signed certificate ./data/traefik/certs/self-signed/cert.pem ; which you
# can add to your Pirogue's operating system trust store) so your pirogue will be able to connect to colander using
# https and this certificate (see below)
# This compose file will also expose Traefik's dashboard on port http://localhost:8080
docker compose -f compose-dev.yml up -d
```

This will :

- create a self signed certificate (allowing you to use pirogue tooling to connect to colander without requiring a certificate
  issued by a Certificate Authority).
- start the whole stack (colander, threatr, mandolin, etc.)
  - you'll get access to:
    - [Colander](http://localhost:5000)
    - [Threatr](http://localhost:5001)
    - [Cyberchef](http://localhost:5002)
    - [Traefik dashboard](http://localhost:8080/)

Note: those tools can also be accessed through their respectie hostnames (if you add them to your `/etc/hosts` file):
  - [Colander](https://colander.local:4443/)
  - [Threatr](https://threatr.local:4443/)
  - [Cyberchef](https://cyberchef.local:4443/)
  - [Traefik dashboard](http://traefik.local:8080/)

### Creating a Threatr integration for your colander

Below are the quick steps to connect your Colander to yoyr Threatr
(this is also [documented on our website](https://pts-project.org/docs/colander/deployment/#connecting-colander-to-threatr)):

- 1. Creating super users on both tools:
  ```bash
  docker compose -f compose-dev.yml exec colander-front /entrypoint python manage.py createsuperuser
  docker compose -f compose-dev.yml exec threatr-front /entrypoint python manage.py createsuperuser
  ```
- 2. Creating a simple user `colander-user` in threatr (via [https://threatr.local/admin](https://threatr.local/admin))
- 3. Creating an API key for `colander-user` in threatr (via [https://threatr.local/admin](https://threatr.local/admin))
- 4. Adding a Backend Credential in threatr with the following value (replace `thetoken` by the token you just created in threatr):
  `{ "api_key": "thetoken"}`

**note:** You can also run `docker compose up -d` but you'll not get access to Traefik's dashboard nor generate a self signed certificate.
`compose.yml` is the base file can be used as a base for production deployments. `compose-dev.yml` overrides some of the service definitions
of the `compose.yml`.
