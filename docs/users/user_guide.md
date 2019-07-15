***Work in progress...***

# Index of contents

*to be defined...*

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

1. #### Stop the current stack

   `rapydo remove`

   

2. #### Switch your git branch to the new version X

   `git fetch && git checkout X`

   

3. #### Upgrade your RAPyDo controller

   `rapydo install --git auto`

By setting the *auto* flag,  RAPyDo will understand by itself which version is required for YOUR_PROJECT.

If your currently installed version is too old, the *auto* flag could not be supported. In this case you can understand the required version by using: `rapydo version` and then upgrade the controller with the command: `rapydo install --git VERSION`.

Otherwise you can install the latest version with pip and then use the *auto* flag:

`pip3 install --upgrade rapydo-controller && rapydo install --git auto`



4. #### Reinitialize your project

`rapydo init`

This step will verify your submodules and will switch them to the correct branch if required.



5. #### Update your submodules

`rapydo update --rebuild`

To skip updates of your main branch and only update your submodules: `rapydo update -i main --rebuild`

By using the *rebuild* flag the update procedure will automatically verify which builds need to be recreated. This option will ensure the minimum amount of time to update the system. As alternative you can perform a complete rebuild of your images once updated:

`rapydo update && rapydo build -rf`



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