***Work in progress...***

# Index of contents

[TOC]

# Production Mode

To switch your RAPyDo deployment from debug to production mode you can use the --mode production options, e.g. `rapydo --mode production start`.

To ensure the option is always activated you can save it in a `.projectrc` file by replacing:

`mode: debug` with `mode: production`

In production mode an additional container based on [NGINX](https://www.nginx.com/) is added to your stack. NGINX is a reverse proxy ensuring security and additional performances to your project. Furthermore it support SSL certificates to enable HTTPS connections.

# SSL Certificates

RAPyDo supports [Let's Encrypt](https://letsencrypt.org/) to automatically deploy SSL certificates. Once started your project in production mode you can request for a SSL certificate with the following command:

```
rapydo ssl-certificate
```

To let this command properly works please very that:

1. your stack is already started (with rapydo start)
2. you configured a correct hostname in your .projectrc

Let's Encrypt certificates expire in 90 days, you can renew them by executing again the `rapydo ssl-certificate` command.

## Automatic certificate renew by using crontab

To make sure your certificate is always up-to-date you can setup a cron job to automatize the certificate renew. You can configure crontab to perform this work for you.

Crontab have some limitations due to the simplified environment used to execute commands, to overcome that limitations you have to:

1. provide absolute path to your rapydo executable (probably `/usr/local/bin/rapydo`)
2. set `COMPOSE_INTERACTIVE_NO_CLI=1` to prevent Compose to use the Docker CLI for interactive `run` and `exec`operations.
3. enable `--no-tty` flag to disable pseudo-tty allocation (by default docker-compose run                         allocates a TTY, not available from crontab)
4. you will haven't access to the command output. If you desire, your can redirect the output on a file

The following crontab entry is able to renew the SSL certificate every Monday at 00:00 AM

```
0 0 * * 1 cd /your/project/path && COMPOSE_INTERACTIVE_NO_CLI=1 /usr/local/bin/rapydo ssl-certificate --no-tty > ~/cron.log 2>&1 
```

[Back to the index](#index-of-contents)



# Upgrade to a new version

We assume that you already have a working deployment of your project (let's name it YOUR_PROJECT) based on version X and you want to upgrade to the new version Y.

1. #### stop the current stack

   `rapydo remove`

2. #### switch your git branch to the new version X

   `git fetch && git checkout X`

3. #### upgrade / update your RAPyDo controller

Probably the new version also requires a new version of RAPyDo. You can verify it by using the following command:

`rapydo version`

This command will give you in output:

​	a. the installed RAPyDo version

​	b. the version of YOUR_PROJECT

​	c. the required RAPyDo version

In this way you will be able to know if a new RAPyDo version is required or not and if you already meet the requisite.



If the **version requirement is unmet** the output will be something like this:

> rapydo: 0.6.6	sci: 0.6.6	required rapydo: 0.6.7
>
> This project is not compatible with the current rapydo version (0.6.6)
>
> Please upgrade rapydo to version 0.6.7 or modify this project
>
> rapydo install --git 0.6.7

You can follow the suggestion or install the new controller by using the *auto* flag `rapydo install --git auto`.

Continue this guide by following step 4 (Upgrade your submodules) 

If you **already have the required RAPyDo** version the output will be something like this:

> rapydo: 0.6.7	YOUR_PROJECT: Y	required rapydo: 0.6.7

Probably you want to upgrade the controller anyway:

`rapydo install --git RAPYDO_VERSION`

You can also use the *auto* flag to let RAPyDo understand by itself which version is required :

`rapydo install --git auto`

Continue this guide by following step 5 (Update your submodules)



4. #### Upgrade your submodules (alternative to step 5)

This step is only required if you upgraded RAPyDo to a new version, in this case you have to switch the version of all the submodules. To switch all your submodules to the new version:

`rapydo init`

This step will require some time since the new version will require new builds. You can skip the building phase here an postpone it on the next step by using the *no-build* flag (`rapydo init --no-build`)



5. #### Update your submodules (alternative to step 4)

This step is only required if you DIDN'T upgrade your RAPyDo version because the new version or YOUR_PROJECT do not require a new version of RAPyDo. In this case you will have to update your submodules and your project branch:

`rapydo update`

To update your submodules only: `rapydo update -i main`

Consider the opportunity to rebuild your containers build with: `rapydo build`



6. #### Upgrade completed

Your upgrade procedure is completed, your are now able to restart your stack

`rapydo start`

Please not that this step could require new builds and take some time (first time only)

[Back to the index](#index-of-contents)

## Known issues post-upgrade

- RAPyDo 0.6.7 upgraded the PostgreSQL version from 10.7 to 11.4. Databases created with psq10 are not compatible with psq11 and your container will fail to start with the following error:

  > FATAL:  database files are incompatible with server
  >
  > DETAIL:  The data directory was initialized by PostgreSQL version 10, which is not compatible with this version 11.4.

  You have two options to fix the error:

  - If your database is only used to store sessions you can simply destroy the database volume and re-initialize it

    1. stop the stack:  `rapydo remove`

    2. delete the sql data volume: `docker volume rm YOUR_PROJECT_sqldata`

    3. start the stack: `rapydo start`
4. enter your backend container: `rapydo shell backend`
    5. re-initialize the database: `restapi init`
  6. exit the backend container: `exit`


​    If you cannot lose your data, please refer to the [Official Upgrading Guide](https://www.postgresql.org/docs/11/upgrading.html)

[Back to the index](#index-of-contents)