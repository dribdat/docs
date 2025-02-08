Installation

---
<img align="right" src="images/logo12.png" width="128">

The following section explains installation options, followed by environment variables you can use to tweak your installation. See also the [README](https://github.com/dribdat/dribdat#quickstart) guide.

# Overview

The core of Dribdat is developed in Python, with an API based on open standards: Linked Data ([JSON-LD](https://json.everyhack.day)), Web-friendly metadata ([Schema.org](https://schema.org/Hackathon)), data packages ([Frictionless Data](https://frictionlessdata.io)), and web authentication ([OAuth](https://oauth.net)). A range of integrations are available to enable sending e-mails, uploading files, or activating AI coaches.

There are several frontends available, from the default web site in Bootstrap, to a Vue.js-based Single Page App ([Backboard](https://github.com/dribdat/backboard)), an older multiplatform chatbot ([Dridbot](https://github.com/dribdat/dridbot)) and a new Tailwind-based dashboard ([Rustboard](https://github.com/dribdat/rustboard)).

With all these options, you might be wondering: what is a good way to start? This page should help you with a basic installation of Dribdat in a short time. 

Please keep in mind that our [Open Collective](https://opencollective.com/dribdat) is a great way to contribute or get additional support from the maintainers!

## Cloud scripts

The installation of dribdat on some cloud providers has been facilitated with quick-deploy scripts.
See [Configuration](#Configuration) below for a list of variables you can set to customize your instance.

<a title="Deploy on Heroku" target="_blank" href="https://heroku.com/deploy?template=https://github.com/dribdat/dribdat"><img src="https://www.herokucdn.com/deploy/button.svg" width="25%"> <a title="Deploy with Vercel" href="https://vercel.com/new/clone?repository-url=https://github.com/dribdat/dribdat" target="_blank"><img src="https://vercel.com/button" width="25%"></a> <a title="Deploy with Akamai" target="_blank" href="https://cloud.linode.com/stackscripts/community?query=dribdat"><img src="https://assets.linode.com/akamai-logo.svg" width="25%"></a>

## With Docker

Containerized Docker builds are available at [Docker Hub](https://hub.docker.com/r/dribdat/dribdat). To deploy dribdat using a local [Docker](https://www.docker.com/) or [Podman](https://docs.podman.io/en/latest/index.html) build, use the included `docker-compose.yml` file as a starting point. This, by default, persists the PostgreSQL database outside the container, on the local filesystem in the `.db` folder.

## With Docker Compose

We also provide a couple of other Docker Compose configurations: for example, you can instantiate a lightweight SQLite version like this:

`chmod o+w data/dribdat.db;docker compose --file docker-compose.sqlite.yml up --detach`

For a first-time setup, you may need to perform the initial migrations as follows:

`docker compose run --rm dribdat ./release.sh`

At this point you should be ready to start with Docker Compose:

`docker compose up -d`

See Configuration below for a list of variables you can set to customize your instance.

## With Ansible

Use the [dribdat Ansible role](https://ansible.build/roles/dribdat/) for a straightforward production deployment using [Ansible](https://docs.ansible.com/). Thanks to Mint Systems for maintaining these scripts.

See [Configuration](#Configuration) below for a list of variables you can set to customize your instance.

## From source

Details on starting the application directly with Python are detailed in the [Developer guide](contribute). You will still want to refer to the [Configuration](#Configuration) section below.

# Configuration

Optimize your dribdat instance with the following **environment variables** in production:

* `TIME_ZONE` - set if your event is not in UTC time (e.g. "Europe/Zurich" - see [pytz docs](https://pythonhosted.org/pytz/)).
* `SERVER_URL` - fully qualified domain name where the site is hosted.
* `DATABASE_URL` - connects to PostgreSQL or another database via `postgresql://username:password@...` (in Heroku this is set automatically)
* `DRIBDAT_SECRET` - a long scary string for hashing your passwords - in Heroku this is set automatically.
* `DRIBDAT_ENV` - 'dev' to enable debugging, 'prod' to optimise for production.

## Features

The following environment variables can be used to toggle **application features**:

* `DRIBDAT_CLOCK` - use 'up' or 'down' to change the position, or 'off' to hide the countdown clock.
* `DRIBDAT_THEME` - can be set to one of the [Bootswatch themes](https://bootswatch.com/).
* `DRIBDAT_STYLE` - provide the address to a CSS stylesheet for custom global styles.
* `DRIBDAT_STAGE` - provide the address to a YAML configuration for custom global [stages](organiser#stages).
* `DRIBDAT_APIKEY` - a secret key for connecting bots with write access to the remote [API](#api).
* `DRIBDAT_ALLOW_LOGINS` - set to False to hide the login, so new users can only log into this server via SSO or by email.
* `DRIBDAT_NOT_REGISTER` - set to True to hide the registration, so new users can only join this server via SSO or invite.
* `DRIBDAT_USER_APPROVE` - set to True so that any new non-SSO accounts are inactive until approved by an admin.
* `DRIBDAT_ALLOW_EVENTS` - set to True to allow regular users to start new events, which admins can promote by un-hiding from the home page.
* `DRIBDAT_SOCIAL_LINKS` - set to False to hide automatic social network links on the site.

## Statistics

Support for **Web analytics** can be configured using one of the following variables:

* `ANALYTICS_FATHOM` ([Fathom](https://usefathom.com/))
* `ANALYTICS_SIMPLE` ([Simple Analytics](https://simpleanalytics.com))
* `ANALYTICS_GOOGLE` (starts with "UA-...")
* `ANALYTICS_HREF` - an optional link in the footer to a public dashboard for your analytics.

If you are required by law to use a cookie warning or banner, you can add this through your community code configuration.

## Mail

If you would like people to be able to activate their accounts and reset passwords, you can connect to an SMTP mailing service (use Mailgun, or any other). Note that this is not really needed when you use OAuth (see next session) and disable registration completely.

* `MAIL_SERVER` - just the domain name of your server.
* `MAIL_PORT` - defaults to 25.
* `MAIL_USERNAME` - the user name of your service.
* `MAIL_PASSWORD` - the password to your service.
* `MAIL_DEFAULT_SENDER` - a required reply-to address for e-mails.
* `MAIL_USE_TLS` - require a secure (TLS) connection when talking to the SMTP server.
* `MAIL_USE_SSL` - require a secure (SSL) connection when talking to the SMTP server.

## Authentication

OAuth 2.0 support for **Single Sign-On** (SSO) is currently available using [Flask Dance](https://flask-dance.readthedocs.io/), and requires SSL to be enabled (using `SERVER_SSL`=1 in production or `OAUTHLIB_INSECURE_TRANSPORT` in development). The following providers are supported:

- `mattermost` - [Mattermost](https://developers.mattermost.com/integrate/apps/authentication/oauth2/)
- `hitobito` - [Hitobito](https://github.com/hitobito/hitobito/blob/master/doc/development/08_oauth.md)
- `github`- [GitHub](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app)
- `slack` - [Slack](https://api.slack.com/authentication/oauth-v2)
- `azure` - [Microsoft Azure](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- `oauth2` - [Generic OAuth 2.0 providers](https://oauth.net/2/) (Zitadel, Auth0, Keycloak, ..)

Register your app with the provider, and set the following variables:

* `OAUTH_TYPE` - one of the supported providers (see above)
* `OAUTH_ID` - the Client ID of your app.
* `OAUTH_SECRET` - the Client Secret of your app.
* `OAUTH_DOMAIN` - OAuth/Mattermost/Hitobito domain, Slack subdomain, or Azure tenant.
* `OAUTH_SCOPE` - (optional) comma-delimited Scope, if not the default (`email,profile,openid`)
* `OAUTH_USERINFO` - (optional) location of userinfo endpoint relative to the domain.
* `OAUTH_SKIP_LOGIN` - (optional) users should go directly to external login screen.
* `OAUTH_LINK_REGISTER` - (optional) a registration link to your SSO platform.
* `OAUTH_HELP_REGISTER` - (optional) a short text for the login page.

You may then wish to disable non-SSO logins using `DRIBDAT_ALLOW_LOGINS` (if an e-mail server is configured, they can still use that), and registrations with `DRIBDAT_NOT_REGISTER`

E-mail activation or moderation (activation in admin area) of non-SSO accounts can be forced with `DRIBDAT_USER_APPROVE` 

You can find more advice in the [Troubleshooting](trouble#need-help-setting-up-sso) guide.

## Spam shield

To try to reduce spam issues on your Dribdat instance, we encourage you to use an OAuth provider (as above) with competency in this area. Basic form validation using [Google Recaptcha](https://developers.google.com/recaptcha) and compatible providers like [Friendly Captcha](https://developer.friendlycaptcha.com/docs/v2/guides/migrating-from-recaptcha), is available:

* `RECAPTCHA_PUBLIC_KEY` - A public key.
* `RECAPTCHA_PRIVATE_KEY` - A private key.
* `RECAPTCHA_API_SERVER` - (optional) Specify your Recaptcha API server.
* `RECAPTCHA_PARAMETERS` - (optional) A dict of JavaScript (api.js) parameters.
* `RECAPTCHA_DATA_ATTRS` - (optional) A dict of [data attributes](https://developers.google.com/recaptcha/docs/display#javascript_resource_apijs_parameters).
* `RECAPTCHA_VERIFY_SERVER` - (optional) The remote API of your alternative Captcha service.
* `RECAPTCHA_SCRIPT` - (optional) The script that is used by your alternative Captcha service.

## File storage

For **uploading images** and other files directly within dribdat, you can configure S3 through Amazon and compatible providers:

* `S3_KEY` - the access key (20 characters, all caps)
* `S3_SECRET` - the generated secret (long, mixed case)
* `S3_BUCKET` - the name of your S3 bucket.
* `S3_REGION` - defaults to 'eu-west-1'.
* `S3_FOLDER` - skip unless you want to store to a subfolder.
* `S3_HTTPS` - URL for web access to your bucket's public files.
* `S3_ENDPOINT` - alternative endpoint for self-hosted Object Storage.
* `MAX_CONTENT_LENGTH` - defaults to 1048576 bytes (1 MB) file size.

See example connection in the [Troubleshooting](trouble#file-storage-example) guide.

Due to the use of the [boto3](https://github.com/boto/boto3/) library for S3 support, there is a dependency on OpenSSL via awscrt. If you use these features, please note that the product includes cryptographic software written by Eric Young (eay@cryptsoft.com) and Tim Hudson (tjh@cryptsoft.com).

## Large language model

If you would like to enable automated challenge and AI-enhanced project suggestions, you can connect OpenAI or compatible service, for example the API endpoint of [LM Studio](https://github.com/lmstudio-ai), or the [LiteLLM](https://docs.litellm.ai/) proxy.

- `LLM_BASE_URL` - if left blank, this uses the production OpenAI endpoint
- `LLM_API_KEY` - (required) the API key of an account with your LLM provider
- `LLM_MODEL` - set to your choice of model, e.g. "gpt-3.5-turbo" - visible to users

## Advanced server settings

These parameters can be used to improve the **production setup**:

* `SERVER_SSL` - redirect all visitors to HTTPS, applying [CSP rules](https://developers.google.com/web/fundamentals/security/csp).
* `SERVER_PROXY` - set to True to use a [standalone proxy](https://flask.palletsprojects.com/en/2.0.x/deploying/wsgi-standalone/#proxy-setups) and static files server - do not use if you already have [a proxy set up](#using-a-proxy-server).
* `SERVER_CORS` - set to False to disable the [CORS whitelist](https://flask.palletsprojects.com/en/2.0.x/deploying/wsgi-standalone/#proxy-setups) for external API access.
* `CSP_DIRECTIVES` - configure content security policy - see [Talisman docs](https://github.com/GoogleCloudPlatform/flask-talisman#content-security-policy).
* `CACHE_TYPE` - speed up the site with Redis or Memcache - see [Flask-Caching](https://flask-caching.readthedocs.io/en/latest/index.html#configuring-flask-caching).

# Tips for your deployment

Here are some additional instructions for your installation. See also the [troubleshooting](trouble) and [contributing](contribute) docs.

## Custom default content

To customize some of the default content, you can edit the template include files in the folder `dribdat/templates/includes`, for example you will find there the default [quickstart.md](https://github.com/dribdat/dribdat/blob/main/dribdat/templates/includes/quickstart.md) and [stages.yaml](https://github.com/dribdat/dribdat/blob/main/dribdat/templates/includes/stages.yaml) definitions. Make sure your changes will not be overwritten is you are using ephemeral storage (e.g. Heroku) for your deployment.

## Using a proxy server

We encourage the use of [Gunicorn](https://flask.palletsprojects.com/en/2.0.x/deploying/wsgi-standalone/#) to run the application in production. Here is an example with 4 workers:

`gunicorn -w 4 -b 127.0.0.1:5000 "dribdat.app:init_app()"`

A web proxy server is typically also used in production to optimize your deployment, add SSL certificates, etc. Here is an example configuration using [nginx](https://nginx.org/) if you are running your application on port 5000:

```
upstream dribdat-cluster {
  server localhost:5000;
}
server {
  listen 80;
  server_name my.dribdat.net;

  # File size limit for uploads
  client_max_body_size 10m;
  keepalive_timeout 0;
  tcp_nopush on;
  tcp_nodelay on;

  # Configure compression
  gzip on; gzip_vary on;
  gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  location / {
    # In case archived URLs were bookmarked
    rewrite ^(.*).html$ $1;
    # Set up the Proxy
    proxy_redirect off;
    proxy_pass http://dribdat-cluster;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location /static {
    # To host assets directly from Nginx (if your files are in /srv/dribdat)
    alias /srv/dribdat/dribdat/static;
    expires 2d;
  }
}
```
