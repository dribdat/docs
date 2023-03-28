This document contains additional information on deploying dribdat.

# Deployment guide

The following section details environment variables you can add to tweak your installation. See also the [README](https://github.com/dribdat/dribdat#quickstart) guide, and our new frontend microservice [Backboard](https://github.com/dribdat/backboard).

## With Python

Details on starting the application directly with Python are detailed in the [Developer guide](contribute). You will still want to refer to the [Configuration](#Configuration) section below.

## With Ansible

Use the [dribdat Ansible role](https://ansible.build/roles/dribdat/) for a straightforward production deployment using [Ansible](https://docs.ansible.com/).

## With Docker Compose

To deploy dribdat using [Docker](https://www.docker.com/) or [Podman](https://docs.podman.io/en/latest/index.html), use the included `docker-compose.yml` file as a starting point. This, by default, persists the PostgreSQL database outside the container, on the local filesystem in the `.db` folder.

For a first-time setup, perform the initial migrations as follows:

`docker-compose run --rm dribdat ./release.sh`

At this point you should be ready to start with Docker Compose:

`docker-compose up -d`

## With cloud hosting

We have battle-tested and tweaked installation of dribdat with the following cloud providers:

[![Deploy on Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/dribdat/dribdat)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/dribdat/dribdat)

[![Deploy with Linode](https://www.linode.com/wp-content/themes/linode-website-theme/images/linode-akamai-logo.svg?ver=1663187393)](https://cloud.linode.com/stackscripts/community?query=dribdat)

# Configuration

Optimize your dribdat instance with the following environment variables in production:

* `TIME_ZONE` - set if your event is not in UTC time (e.g. "Europe/Zurich" - see [pytz docs](https://pythonhosted.org/pytz/)).
* `SERVER_URL` - fully qualified domain name where the site is hosted.
* `DATABASE_URL` - connects to PostgreSQL or another database via `postgresql://username:password@...` (in Heroku this is set automatically)
* `DRIBDAT_SECRET` - a long scary string for hashing your passwords - in Heroku this is set automatically.
* `DRIBDAT_ENV` - 'dev' to enable debugging, 'prod' to optimise for production.

## Server settings

These parameters can be used to improve the **production setup**:

* `SERVER_SSL` - redirect all visitors to HTTPS, applying [CSP rules](https://developers.google.com/web/fundamentals/security/csp).
* `SERVER_PROXY` - set to True to use a [standalone proxy](https://flask.palletsprojects.com/en/2.0.x/deploying/wsgi-standalone/#proxy-setups) and static files server - do not use if you already have [a proxy set up](#using-a-proxy-server).
* `SERVER_CORS` - set to False to disable the [CORS whitelist](https://flask.palletsprojects.com/en/2.0.x/deploying/wsgi-standalone/#proxy-setups) for external API access.
* `CSP_DIRECTIVES` - configure content security policy - see [Talisman docs](https://github.com/GoogleCloudPlatform/flask-talisman#content-security-policy).
* `CACHE_TYPE` - speed up the site with Redis or Memcache - see [Flask-Caching](https://flask-caching.readthedocs.io/en/latest/index.html#configuring-flask-caching).

## Features

The following options can be used to toggle **application features**:

* `DRIBDAT_THEME` - can be set to one of the [Bootswatch themes](https://bootswatch.com/).
* `DRIBDAT_STYLE` - provide the address to a CSS stylesheet for custom global styles.
* `DRIBDAT_CLOCK` - use 'up' or 'down' to change the position, or 'off' to hide the countdown clock.
* `DRIBDAT_APIKEY` - a secret key for connecting bots with write access to the remote [API](#api).
* `DRIBDAT_USER_APPROVE` - set to True so that any new non-SSO accounts are inactive until approved by an admin.
* `DRIBDAT_NOT_REGISTER` - set to True to hide the registration, so new users can only join this server via SSO.
* `DRIBDAT_ALLOW_EVENTS` - set to True to allow regular users to start new events, which admins can edit to make visible on the home page.
* `DRIBDAT_SOCIAL_LINKS` - set to False to hide automatic social network links (Twitter etc.) on the site.

## Statistics

Support for **Web analytics** can be configured using one of the following variables:

* `ANALYTICS_FATHOM` ([Fathom](https://usefathom.com/) - with optional `ANALYTICS_FATHOM_SITE` if you use a custom site)
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

OAuth 2.0 support for **Single Sign-On** (SSO) is currently available using [Flask Dance](https://flask-dance.readthedocs.io/), and requires SSL to be enabled (using `SERVER_SSL`=1 in production or `OAUTHLIB_INSECURE_TRANSPORT` in development). You may wish to disable non-SSO logins using `DRIBDAT_NOT_REGISTER`.

Register your app with the provider, and set the following variables:

* `OAUTH_TYPE` - a supported provider: 'mattermost', 'slack', 'github', 'azure', 'auth0'
* `OAUTH_ID` - the Client ID of your app.
* `OAUTH_SECRET` - the Client Secret of your app.
* `OAUTH_DOMAIN` - Slack subdomain, Auth0/Mattermost domain, or Azure tenant.
* `OAUTH_SKIP_LOGIN` - (optional) users should go directly to external login screen.
* `OAUTH_LINK_REGISTER` - (optional) an external registration link.

You can find more advice in the [Troubleshooting](trouble#need-help-setting-up-sso) guide.

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

## Custom content

To customize some of the default content, you can edit the template include files in the folder `dribdat/templates/includes`, for example you will find there the default [quickstart.md](https://github.com/dribdat/dribdat/blob/main/dribdat/templates/includes/quickstart.md) and [stages.yaml](https://github.com/dribdat/dribdat/blob/main/dribdat/templates/includes/stages.yaml) definitions. 

Make sure your changes will not be overwritten is you are using ephemeral storage (e.g. Heroku) for your deployment. In a future release we may provide an environmental variable for configurations hosted elsewhere. For now, copy the files from a secure location during a deployment step.

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
    # To host assets directly from Nginx
    expires 2d;
    alias /srv/dribdat/dribdat/static;
  }
}
```
