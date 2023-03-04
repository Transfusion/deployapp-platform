# DeployApp 

## Running (Development)
1. Install [`docker-sync`](http://docker-sync.io/) and [`mkcert`](https://github.com/FiloSottile/mkcert)
2. Clone this repository
3. Edit [`docker-compose-dev.yml`](https://github.com/Transfusion/deployapp-platform/blob/dev/docker-compose-dev.yml)
- Create a [GitHub Personal access token](https://github.com/settings/tokens/) with the `read:packages` permission.
- Set `GPR_USERNAME` to your own GH username and `GPR_PAT` accordingly.
4. Point a domain to your machine for the OAuth2 social login and iOS OTA install features to work. The `.local` mDNS special-use domain name works too.
- Generate a TLS cert for your domain. Self-signed works too. Do not change `deployapp.local.key` and `deployapp.local.crt`; they are used in `deployapp-web-config/nginx.conf`.

      cd deployapp-web-config/certs && mkcert -cert-file deployapp.local.crt -key-file deployapp.local.key CHANGE_THIS.local

5. Edit `deployapp-backend-config/application-dev.yml`.
- Set `spring.mail.*`. Used during user registration and password reset.
- Set `security.oauth2.client.registration.*`. Used during OAuth2 social login.
  - Create the respective OAuth applications, e.g. a Google Cloud Platform project and a new OAuth 2.0 Client ID.
  - Google does not allow `.local` as a valid TLD in its redirect URLs. https://redirectmeto.com is a workaround.
  - E.g. `https://redirectmeto.com/https://CHANGE_THIS.local:12346/login/oauth2/code/google`
  - Set `client-id`, `client-secret`, and `redirect-uri` for the social providers that you want enabled.

- Add `https://CHANGE_THIS.local:12346` (see Ports Overview below) to the comma-separated `custom_cors.origins`.

6. Edit `deployapp-storage-service-config/application-dev.yml`.
- Add `https://CHANGE_THIS.local:12346` (see Ports Overview below) to the comma-separated `custom_cors.origins`, identical to `deployapp-backend-config/application-dev.yml`.

7. Edit `deployapp-frontend-config/.dev.env`.
- `REACT_APP_BASE_URL` is the root backend endpoint. 
  - Set it to `https://CHANGE_THIS.local:12346/`.
  - In development, we reverse proxy the frontend through NGINX under the same port for simplicity (the concept of subdomains doesn't apply to `.local` mDNS, etc.)
  - In production, `dapp-backend` and `dapp-storage-service` are reverse proxied under their own subdomain; `https://api.deploy.plan.ovh`.
- `REACT_APP_OAUTH_REDIRECT_BASE_URL` is the root frontend endpoint.
  - Set it to `https://CHANGE_THIS.local:12346/` for the same reasons above.
- Set `REACT_APP_GOOGLE_AUTH_URL` to the value of `REACT_APP_BASE_URL` + `/oauth2/authorize/google?redirect_uri=`.
- Set `REACT_APP_GITHUB_AUTH_URL` in the same manner.

8. `docker-sync start`
9. `docker-compose compose -f docker-compose-dev.yml up`

Follow [the `mkcert` instructions](https://github.com/FiloSottile/mkcert/blob/master/README.md) to install the CA into your system trust store or that of your other devices.

DeployApp will be available at `https://CHANGE_THIS.local:12346`.

## Running (Release)

TBD

## Architecture
TBD

## Ports Overview

RabbitMQ (`dapp-rabbitmq`)
- `15672:5672`
- `25672:15672` Web management console.
  - Default creds: `guest:guest`

Redis (`dapp-redis`)
- `16379:6379`
  - Default password: `password`

PostgreSQL (`dapp-db`)
- `15432:5432`
  - Default creds: `postgres:password`
  - Default databases: `deployapp_dev_1,deployapp_storage_dev_1`

Backend service (`dapp-backend`)
- `18080:8080` HTTP
- `18082:8082` [JobRunr](https://www.jobrunr.io/en/) Dashboard
- `15010:5010` JDWP debug

Storage service (`dapp-storage-service`)
- `19080:8080` HTTP
- `16010:5010` JDWP debug

Frontend (`dapp-frontend`)
- Not exposed; listens internally on port `3000`.

NGINX reverse proxy (`dapp-web`)
- `12345:12345` HTTP (for debugging purposes)
- `12346:12346` HTTPS (self-signed)

\* `dapp-frontend` is pointed to `12346/https` by default.
