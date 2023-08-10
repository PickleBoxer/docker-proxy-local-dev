# Setup own CA

## Hosts (optional for custom domains) or local DNS

`%WINDIR%\System32\drivers\etc\hosts`

```
127.0.0.1 local.dev
::1       local.dev
```

## New Docker network

```bash
docker network create traefik_public
docker network create socket_proxy
```

## Certificates

We need to create our own TLS certificates in order to have encryption in transit and properly secure the communication from and to Traefik.


```sh
install -d certs
mkcert -key-file certs/key.pem -cert-file certs/cert.pem local.dev *.local.dev
mkcert -install
```

The generated traefik certificate is a wildcard certificate for `*.local.dev`

## Deployment

Deploy the containers by executing the following command:

```sh
docker-compose up -d
```

## Dashboard

You can access the dashboard by visiting

`https://traefik.local.dev/dashboard/` and the api at `https://traefik.local.dev/api/rawdata`.

## References

- https://dev.to/karvounis/advanced-traefik-configuration-tutorial-tls-dashboard-ping-metrics-authentication-and-more-4doh
- https://github.com/karvounis/traefik-tutorial-docker-compose-files