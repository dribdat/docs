# Frequently asked

Guidance to troubleshooting common issues and questions in Dribdat can be found here. For less technical questions, there is also a [Q & A for organisers](organiser).
For further technical references, see the [Deployment guide](deploy).

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

---

## F.A.Q

The following questions were originally compiled as part of the projects [DINAcon nomination](https://dinacon.ch/en/dinacon-awards/nominations/).
You can contribute to additional social review of the project at [AlternativeTo](https://alternativeto.net/software/dribdat/).

### *How mature is the project?*

Live and production ready.

### *When was the project started?*

November 2015

### *Under which license is your project licensed?*

OSI/FSD-approved free software license (MIT)

### *Does one need to sign a contributor agreement?*

No

### *How many different contributors did contribute to the project within the last 12 months?*

5

### *How many commits did your project receive within the last 12 months? Projects not having a version control system (e.g. Open Data projects or similar) can also specify changesets or similar.*

393

### *Was this project forked (split into different communities) in the past or is this project a fork of an other project? Please describe the situation (was it a friendly/unfriendly fork, what was the reason, what is the current state of the projects that are/were involved).*

No

### *If your project is based on a (public) GitLab instance, on Github, on Bitbucket, etc. that offers a metric like "followers" or "likes", what is the metric of your project?*

38 stars, 17 forks

### *Does the community meet in "real life" on a regular base (e.g. yearly, monthly, ...)? How many people do meet at such meetings?*

Yes. We meet at least twice a year at events where we use, and further develop, this platform. There is a small group of people that has directly influenced, and continues to take an interest in, the course of this project. We have several online locations to share updates (notably on [Open Collective](https://opencollective.com/dribdat/updates), ~~Mattermost, Discord and Slack~~).

### *Do you have other metrics that you would like to share with us, which help us to understand how successful your project is?*

Used at 50+ hackathons. The single largest installation has 750+ users.

### *Does your project have a non-coder community e.g. UX designer, translator, marketing, etc.? What kind of non-coder people are involved in your project?*

Yes. Hackathon participations are not just coders, and non-technical users who would like to discover open source activities are an important prerogative for this project. The process of running creative events like hackathons, combining our experience in code, is something we have aimed to build into all parts of the tool to make the event format accessible to an even wider public.

### *Does your project offer something like "easy hacks" to make it easy to get a foot into the project?*

Yes, there is a getting started page which can also be customized by the organisers. We have a community instance, where people can start their own events. To run your own instance of dribdat, there are clear installation instructions to follow.

### *Please provide the page where we can find more information about the easy hacks.*

- For deployment see the [deployment guide](/deploy).
- For usage notes, see the [user handbook](/usage).
- If you have question, drop them in [our forum](https://github.com/orgs/dribdat/discussions).

### *Do you have people mentoring new contributors and actively helping them to get on board of the project?*

Oleg and other people with experience in using the platform are available via various community channels to new users of dribdat. We regularly mentor new users in getting started. There is also a hosted version of the platform which we can set up to get hackathons going quickly. And a reasonably [active forum](https://forum.schoolofdata.ch/t/make-the-most-of-hackathon-season/167).

### *Does your project undertake specific efforts to make sure, that people regardless of ethnicity, gender, etc. can contribute and participate in the project? Does your project for example have a Code of Conduct or is it in general open and friendly to all human beings? Please explain the current situation.*

The [Hack Code of Conduct](https://hackcodeofconduct.org/), which we have applied to events for the past 3 years, is now in dribdat by default to help facilitate inclusive events.

### *Please describe the business case of your project.*

There are hackathons happening all over the world every weekend. Countless more collaborative online "hacking" events happen every day in companies, institutions and the civil society. Each of them generates interest, activity, networking and new initiatives. While several hackathon platforms contend to "lead the market", we are one of a handful of open source alternatives, most notably [HackDash](https://github.com/impronunciable/hackdash/) and [Sparkboard](https://github.com/sparkboard/sparkboard) - which operate in a SaaS model.

Please visit our [OpenCollective](https://opencollective.com/dribdat), where we are currently focusing our fundraising and transparent budgeting.

### *How relevant is your project in regard to a commercial use?*

Currently we accept sponsoring and distribute it within the project, but do not charge a licensing fee of any kind. There is a strong, recurrent interest from companies in using dashboards similar to this one for tracking internal activities above and beyond hackathon-type events. Several commercial models come to mind: the original motivation for the project came out of a hackathon sponsored by Swisscom, a company that champions innovation culture. We are inspired us to pursue an organic and grassroots business model that benefits a wide variety of "start-up" initiatives.

### *This project is important to the world because....*

Hackathons have become a useful instrument to see critically beyond the veil of pragmatic utility in Information Technologies, and have been embraced by the most disruptive companies and organisations around the world as a vehicle for positive change. By imbuing a software project with the ethics and values of hackathons, we can scale these experiments from our local community to many other corners of the world and many domains of creative collaboration.

### *If this project stops its development and ceases to exist, this would be the impact...*

We wouldn't have our own platform to hack the meta-side of hackathons, and that would be a shame. We could go back to using wikis and repo organisations, with all the constraints and loss of user friendliness that entails.

### *Do you have any usage metrics you can provide which show how succesful your project is? (downloads, visits on website, registered users, etc.)*

In addition to the low thousands of user accounts and hundreds of projects across the different installations (these are visible to administrators in the dashboard), many dribdat instances have "open analytics": scroll down to the bottom of the page (for example, on hack.opendata.ch) and click Analytics.
