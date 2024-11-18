# Frequently asked questions

Guidance to troubleshooting common issues and quesitons in Dribdat can be found here.
For more technical references, see the [README](https://codeberg.org/dribdat/dribdat#dribdat).

---

## User interface

### How to insert the projects and challenges into my own web page

There is an **Embed** button in the event admin which provides you with code for an IFRAME that just contains the hexagrid. If you would like to embed the entire application, and find it more intuitive to hide the navigation, add `?clean=1` to the URL. To also hide the top header, use `?minimal=1`.

The [Backboard](https://codeberg.org/dribdat/backboard) project is our new alternative front-end, that invokes the [dribdat API](#API) to visualize data from the platform.

If your CMS supports RSS feeds, you can also embed the latest activities using this format. Check the About page for the link.

### Navigation is not visible

Some Bootswatch themes do not play well with the *navbar-light* component used in our layout (`nav.html`). Override the styles by hand using the `DRIBDAT_CSS_URL` environment variable.

---

## User management

### How to restore admin access

The first user account on the system gets automatically promoted to admin.

If you are locked out of the administration, run `./manage.py shell` on the console.

You can then promote any user to admin and/or reset the password of a user called "admin" like this:

```
u = User.query.filter(User.username=='admin').first()
u.is_admin = True
u.set_password('Ins@nEl*/c0mpl3x')
u.save()
```

### Need help setting up SSO

To get client keys, go to the [Slack API](https://api.slack.com/apps/), [Azure portal](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), or add the [GitHub App](https://github.com/apps/dribdat) to your account or organization. You can also use [custom OAuth 2](https://flask-dance.readthedocs.io/en/latest/providers.html#custom) provider if you provide all external registration URLs.

Cannot determine SSO callback for app registration? Try this:

`<my server url>/oauth/<my provider>/authorized`

Where the provider is `slack`, `mattermost`, .. as configured in `OAUTH_TYPE`

### Cleaning out inactive users

Open a `manage.py shell` and run a command like this to remove all non-admin, inactive, non-SSO users with zero drib-scores:

```
from dribdat.user.models import User
for pp in User.query.filter_by(is_admin=False, active=False, sso_id=None):
  if pp.activity_count == 0 and len(pp.roles) == 0 and len(pp.posted_challenges()) == 0: pp.delete()
```

---

## Data management

### File storage example

Here is an example of how to configure your own S3 provider (i.e. not Amazon) for serving files:

```
S3_BUCKET=my-bucket-a1bc2d3f4g
S3_FOLDER=dribdat4uri
S3_ENDPOINT=https://my-bucket-a1bc2d3f4g.s3.provider.com
S3_HTTPS=https://my-bucket-a1bc2d3f4g.https.provider.com
S3_KEY=BC7CH3CNMVERYSECRET
S3_SECRET=abCdEfGhIjKlMnOpQr
```

Make sure to provide an HTTPS link, as normally you would like to be able to show uploaded files, not just store them. This may require setting permissions and CORS settings accordingly with your provider. We have tested this set up with Linode Object Storage, Exascale, Bucketeer and others.

### Cannot upgrade database

In local deployment, you will need to upgrade the database using `./manage.py db upgrade`.

On Heroku, a deployment process called **Release** runs automatically. Note also the **Socialize** process which is used for refreshing user profiles.

If you get errors like *ERROR [alembic.env] Can't locate revision identified by 'aa969b4f9f51'*, your migration history is out of sync. You can set `FORCE_MIGRATE` to 1 when you run releases, however changes to the column sizes and other schema details will not be deployed. Instead, it is better to verify the latest schema specifications in the `migrations` folder, fix anything that is out of sync, and then update the alembic version, e.g.:

```
alter table projects alter column webpage_url type character varying(2048);
insert into alembic_version values ('7c3929047190')
```

A handy parameter is `--sql` which shows just the SQL code you can also apply manually to fix your database. See also further instructions in the `force-migrate.sh` script.

### Invalid input value for enum activity_type error

There were issues in upgrading your instance that may require a manual SQL entry. Try running these commands in your `psql` console:

```
ALTER TYPE activity_type ADD VALUE 'boost';
ALTER TYPE activity_type ADD VALUE 'review';
```

### No profile images after updating

Run `manage.py socialize users` to restore the profile images. This is due to a change in the way they are stored, to make the profile more flexible.

---

## Developer environment

### Test locally using SSL

Some development scenarios and OAuth testing requires SSL. To use this in development with self-signed certificates (you will get a browser warning), start the server with `./manage.py run --cert=adhoc`

You can test SSO providers in this way by adding `OAUTHLIB_INSECURE_TRANSPORT=true` to your environment (do not use in production!)

### Installation on Alpine Linux

The project has so far mostly been developed on Arch, Fedora and Ubuntu Linux. Users on Alpine, BSD and other distributions are welcome to share their experience with us in the Issues. Some additional system packages are needed for a successful local (non-Docker) deployment.

E.g. for Alpine or Ubuntu (apt instead of apk):

```
apk add libxml2-dev libxslt-dev libffi-dev rust cargo
```

### Error compiling misaka

You are missing development headers for Python. For example, in Fedora Linux run:

```
sudo dnf install libffi-devel python3-devel gcc
```

On Ubuntu (`apt`) or Alpine Linux (`apk`), the command will look more like this:

```
sudo apk add libffi-dev python3-dev gcc
```

### Updating the requirements file

Install the [Poetry Export plugin](https://github.com/python-poetry/poetry-plugin-export) (this may not be necessary in the near future), then run:

`poetry export -f "requirements.txt" --output "requirements/prod.txt" --without-hashes`
