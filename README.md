# Docker Local Development with Traefik

Traefik can be configured to accept incoming HTTPS connections in order to terminate the SSL connections. It can be configured to use an ACME provider (like Let's Encrypt or [Step-Ca](https://smallstep.com/certificates/)) for automatic certificate generation.
Or we can create our own CA and Traefik certificates and configure Traefik to use them.

## Prerequisites

Before you begin, make sure you have the following installed on your system:

- Docker
- Docker Compose

## Variant 1: Self-Signed Certificate

1. Navigate to the `traefik-own-ca` directory:

    ```bash
    cd traefik-own-ca
    ```

2. Generate a self-signed certificate and key preferably using mkcert or any other tool of your choice. You can also use the pre-generated certificate and key in the `certs` directory.

3. Start the Traefik container:

    ```bash
    docker-compose up -d
    ```

## Variant 2: Automatic Certificate Signing with Step-Ca

1. Navigate to the `traefik-step-ca` directory:

    ```bash
    cd traefik-step-ca
    ```

2. Follow the instructions in the subfolder

3. Start the Traefik container:

    ```bash
    docker-compose up -d
    ```

## Conclusion

You should now have a working Traefik setup for local development using either a self-signed certificate or the Step-Ca library for automatic certificate signing.
