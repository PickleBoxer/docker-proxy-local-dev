# Setup step-ca

## Hosts (optional for custom domains) or local DNS

`%WINDIR%\System32\drivers\etc\hosts`

```
127.0.0.1 docker.local
::1	docker.local
```

## New Docker network

```bash
docker network create traefik-net
```

## init step-ca (optional)

Comment out the following lines in `docker-compose.yml`:

```yaml
environment:
      DOCKER_STEPCA_INIT_NAME: Smallstep
      DOCKER_STEPCA_INIT_DNS_NAMES: step-ca.docker.local,localhost
      DOCKER_STEPCA_INIT_PROVISIONER_NAME: admin
      DOCKER_STEPCA_INIT_PASSWORD: pass123
      DOCKER_STEPCA_INIT_ACME: "true" # initialize acme provider
```

```sh
docker run --rm -it -v "$PWD/data/step-ca:/home/step" smallstep/step-ca step ca init
```

## Start your step-ca Instance

```sh
docker-compose up -d step-ca
```

```bash
Generating root certificate... done!
Generating intermediate certificate... done!

✔ Root certificate: /home/step/certs/root_ca.crt
✔ Root private key: /home/step/secrets/root_ca_key
✔ Root fingerprint: a69a161dd6804fe65edac4e6f7425caf225bf22a1b5efa783a67c80fb60a3da5
✔ Intermediate certificate: /home/step/certs/intermediate_ca.crt
✔ Intermediate private key: /home/step/secrets/intermediate_ca_key
✔ Database folder: /home/step/db
✔ Default configuration: /home/step/config/defaults.json
✔ Certificate Authority configuration: /home/step/config/ca.json

Your PKI is ready to go. To generate certificates for individual services see 'step help ca'.

FEEDBACK 😍 🍻
  The step utility is not instrumented for usage statistics. It does not phone
  home. But your feedback is extremely valuable. Any information you can provide
  regarding how you’re using `step` helps. Please send us a sentence or two,
  good or bad at feedback@smallstep.com or join GitHub Discussions
  https://github.com/smallstep/certificates/discussions and our Discord 
  https://u.step.sm/discord.
badger YYYY/MM/DD HH:MM:SS INFO: All 0 tables opened in 0s
YYYY/MM/DD HH:MM:SS Starting Smallstep CA/0.24.2 (linux/amd64)
YYYY/MM/DD HH:MM:SS Documentation: https://u.step.sm/docs/ca
YYYY/MM/DD HH:MM:SS Community Discord: https://u.step.sm/discord
YYYY/MM/DD HH:MM:SS Config file: /home/step/config/ca.json
YYYY/MM/DD HH:MM:SS The primary server URL is https://step-ca.docker.local:9000
YYYY/MM/DD HH:MM:SS Root certificates are available at https://step-ca.docker.local:9000/roots.pem
YYYY/MM/DD HH:MM:SS Additional configured hostnames: localhost
YYYY/MM/DD HH:MM:SS X.509 Root Fingerprint: a69a161dd6804fe65edac4e6f7425caf225bf22a1b5efa783a67c80fb60a3da5
YYYY/MM/DD HH:MM:SS Serving HTTPS on :9000 ...

👉 Your CA administrative password is: pass123
🤫 This will only be displayed once.
```

Then, go to https://step-ca.docker.local:9000/health to make sure service is running.

## Enable ACME provisioner

```sh
docker-compose exec step-ca step ca provisioner add acme --type ACME
docker-compose restart
```

## Add CA to your development environment

To interact with step-ca, you'll want to install the step client in your host environment. See our [installation docs](https://smallstep.com/docs/step-cli/installation).

Get fingerprint:

```sh
echo CA_FINGERPRINT=$(docker run  -v <volume>:/home/step smallstep/step-ca step certificate fingerprint /home/step/certs/root_ca.crt)
```

Add CA to host:

```sh
step ca bootstrap --ca-url https://step-ca.docker.local:9000 --install --fingerprint <fingerprint-acquired>
```

All-in-one:

```sh
{
CA_FINGERPRINT=$(docker run  -v traefik-step-ca_step-ca:/home/step smallstep/step-ca step certificate fingerprint /home/step/certs/root_ca.crt)
step ca bootstrap --ca-url https://step-ca.docker.local:9000 --fingerprint $CA_FINGERPRINT --install
}
```

```bash
The root certificate has been saved in /home/user/.step/certs/root_ca.crt.
The authority configuration has been saved in /home/user/.step/config/defaults.json.
Installing the root certificate in the system truststore... [sudo] password for user:
done.
```

This command setup created CA on your computer to be able to acquire certificates, and adds the CA to your computer's trust store.

Check if CA is added to your trust store.

```sh
curl https://step-ca.docker.local:9000/health
```

Create a sample certificate for localhost.

```sh
step ca certificate step-ca.localhost step_ca_localhost.crt step_ca_localhost.key
```

Save a copy of your root CA certificate:

```sh
step ca root root_ca.crt
```

### Run traefik first

```sh
docker-compose up -d traefik
sleep 10
docker-compose up -d whoami
```

## Initializing step-ca from your own CA certificate and key

```sh
docker-compose down
sudo rm -rf ./data
mkdir -p ./data/step-ca/secrets

cp "$(mkcert -CAROOT)/rootCA.pem" ./data/step-ca/
cp "$(mkcert -CAROOT)/rootCA-key.pem" ./data/step-ca/
echo '123456' | tee "$PWD/data/step-ca/secrets/password"

# don't chown on MacOS
sudo chown -R 1000:1000 "$PWD/data/step-ca"

docker-compose run step-ca step ca init \
--root "/home/step/rootCA.pem" \
--key "/home/step/rootCA-key.pem" \
--name "mkcert CA" \
--provisioner "admin" \
--dns "localhost,ca.internal,ca.myhost.local,acme.myhost.local" \
--address ":9000" \
--password-file=/home/step/secrets/password

docker-compose up -d step-ca
docker-compose exec step-ca step ca provisioner add acme --type ACME
docker-compose restart
docker-compose up -d traefik
docker-compose logs -f traefik
```

ref: https://smallstep.com/docs/tutorials/intermediate-ca-new-ca

## References

- https://hub.docker.com/r/smallstep/step-ca
- https://smallstep.com/docs/step-ca/installation