# Catalyze

We're using [Catalyze](https://catalyze.io/) as our platform-as-a-service
vendor. Catalyze uses Heroku buildpacks and is built on top of Amazon Web
Services (AWS), so it should be relatively easy to transition back to Heroku if
needed once [private spaces](https://www.heroku.com/private-spaces) become
HIPAA-compliant.

The potential advantage of Catalyze is that they're laser-focused on
healthcare, so it's possible they'll handle more of the tedium of HIPAA than
Heroku ever would, but we'll see.

## Setup

1. Add the Catalyze git remotes to `.git/config` according to the README of the
   repo you are trying to deploy.

1. Install the [Catalyze CLI](https://github.com/catalyzeio/cli).

   (NOTE: Beware of catalyze-paas-cli, which is out of date.)

1. Add your Catalyze credentials to a `.catalyze` file in the root directory,
   using `.catalyze.example` as a template.

1. Associate yourself with Catalyze. This assumes you are in the repo and have
   access to our Catalyze wrapper script (`bin/catalyze.sh`):

   ```
   $ bin/catalyze.sh associate healthproprod
   $ bin/catalyze.sh associate healthprostaging
   ```

1. Ensure you are logged into the right Catalyze user and have access:

   ```
   $ bin/catalyze.sh associated
   ```

   You should see `healthproprod` and `healthprostaging` in the list. If you
   don't, ask for access.

## Environments

Right now, there are two environments at Catalyze: `healthproprod` and
`healthprostaging`.

To specify an environment when running a command use the `-E` or `--env` flag,
e.g. `bin/catalyze.sh -E healthproprod <do-something>`.

I'd recommend setting your default environment to `healthprostaging` to force
yourself to add additional flags when operating on production. This should help
avoid accidentally running dangerous commands against production. To set your
default environment to staging, use `bin/catalyze.sh default healthprostaging`.

## Services

We have a few services running at Catalyze for each environment:

- `app01`: The core Rails application + React frontend
- `db01`: The production database server
- `onboarding01`: The production onboarding app

## Logging and monitoring

Logging and monitoring information can be accessed through the [Catalyze
dashboard](https://dashboard.catalyze.io/signin). Select Environments ->
healthproprod, then look for "Logging" and "Monitoring" in the first box.

Logs are also accessible through the command line. An example command:

    # view logs for the last 2 hours
    $ bin/catalyze.sh logs "*" --hours=2

See `bin/catalyze.sh logs --help` for full details, but be aware that the
default query of "app*" seems to miss most of the logs we care about.

Detailed monitoring information is not accessible via the command line, but you
can get some high-level stats using `bin/catalyze.sh metrics` and
`bin/catalyze.sh status`.

## Other common commands

### View ENV variables

    $ bin/catalyze.sh vars list

### Set ENV variable

    $ bin/catalyze.sh vars set -v FOO=bar

### Run a Rails console

    $ bin/catalyze.sh console app01 "bundle exec rails console"

(NOTE: This command does not accept `rails c` instead of `rails console`.)

### Connect to Postgres

    $ bin/catalyze.sh console db01

## Debugging Tips

The first time you deploy code to a new environment, Catalyze needs to
configure the application in nginx to receive its intended requests. There are
two common error codes to be aware of:

### 502 Bad Gateway

Catalyze has not yet configured nginx for your application. The solution is to
submit a support ticket and let them know that the code has been deployed.

### 503 Service Unavailable

Your application is configured correctly in nginx, but the container is not
receiving requests. This usually means there's an error starting your
application -- often a runtime error in a Rails initializer or other startup
code. Check the Catalyze logs (see above) to debug the error, fix it, then
redeploy. It's possible that Catalyze could be responsible for 503 errors in
the future, but most of them seem to be on our side so far.
