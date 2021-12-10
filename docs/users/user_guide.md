*User Guide - Table of Contents*

<!--ts-->
   * [User Guide](#user-guide)
      * [Project Configuration](#project-configuration)
         * [Stacks](#stacks)
         * [Services](#services)
         * [Swagger](#swagger)
         * [Containers builds](#containers-builds)
         * [Interfaces](#interfaces)
         * [Multi projects](#multi-projects)
         * [Production mode](#production-mode)
         * [SSL Certificates](#ssl-certificates)
      * [Upgrade to a new version](#upgrade-to-a-new-version)
      * [Known issues post-upgrade](#known-issues-post-upgrade)
         * [PostgreSQL fails to start with RAPyDo 2.2](#postgresql-fails-to-start-with-rapydo-22)
         * [Neo4j fails to start with RAPyDo 2.0](#neo4j-fails-to-start-with-rapydo-20)

<!-- Added by: mdantonio, at: ven 10 dic 2021, 10:51:47, CET -->

<!--te-->


# User Guide

## Project Configuration

Each project has three level of configuration:

- default configuration (defined by RAPyDo it-self)
- project configuration (defined by a developer in `projects/YOUR_PROJECT/project_configuration.yml`)
- instance configuration (defined by the user in a `.projectrc` file)

A .projectrc file (created in the root of the project) can customize most of options by overriding configurations at project and RAPyDo levels

Here a typical `.projectrc` file:



`project: your_project`

`# stack: your_custom_stack`

`hostname: localhost`

`# production: True`

`project_configuration:`
  `variables:`
    `env:`
      `SMTP_ADMIN: your@email.org`

​      `other variabiles...`

### Stacks

In every project there is a set configurations that you would like to use to switch. Even if not the case, there should be at least two modes: 

1. `development` for local development (verbose logs, auto reload and no reverse proxy)
2. `production` for deploying  the project in production (less logs, `gunicorn` and a reverse proxy, all set to use `SSL` for free with Let's Encrypt).

Stacks are just implemented as `YAML` files based on the `docker compose` syntax. You can create as many custom stacks as you need.

The final configuration you will use in every RAPyDo command is based on:

- A RAPyDo base configuration, gathering the best practices on the most commonly used services
- A common custom configuration across different modes in your project (in `projects/YOUR_PROJECT/confs/commons.yml`)
- A custom file dedicated to a single mode  (in `projects/YOUR_PROJECT/confs/YOUR_STACK.yml`)

Default stacks (development and production) are automatically enabled

You can enable production stack by passing --production (or --prod) option to any RAPyDo command or by setting production: True in your .projectrc. You can enable any other custom stack by passing --stack STACK_NAME to any RAPyDo command or by setting stack: STACK_name in your .projectrc.

### Services

RAPyDo is of containers oriented and this means that every service can be easily added to be tested locally. We tested several services in our projects thus they are already integrated:

- PostgreSQL
- MariaDB
- Neo4j
- MongoDB
- Redis
- RabbitMQ
- Celery
- Pure-FTP
- Angular
- Pushpin

More services can be added as long as you can provide a container image (official or not) for it; so basically always!

In a stack (e.g. in production) you may choose to switch to external existing services by simply overriding the SERVICE configuration (HOST, PORT, etc), without changing your endpoint code.

### Swagger

The [OpenAPI standard]() has helped us many times to show to clients the experience of HTTP API services without even using a frontend. This is why every endpoint defined with RAPyDo is fully OpenAPI-compliant and included in a Swagger specification file available at the /api/swagger endpoint.

### Containers builds

All core RAPyDo images are automatically built and pushed on the Docker Hub. You can download all required images by using the `rapydo pull` command. A project can extend a base image (e.g. to install additional libraries required for custom endpoints). Custom images can be built by using the `rapydo build` command.

### Interfaces

A set of interfaces can be launched as containers to help with many services:

- swaggerui
- adminer for PostgreSQL, MySQL/MariaDB and MongoDB
- flower for celery

All interfaces can be executed by using the `rapydo run` command.

### Multi projects

The same repository can host different projects. This action came handy quite a lot of times when a similar project has to be maintained in the same repository by the same people.

If the project is only one it is used as default. To switch projects you can use the `--project` command line parameter or set it inside the `.projectrc` file.

### Production mode

In production mode an additional container based on [NGINX](https://www.nginx.com/) is automatically included to your stack. NGINX is a reverse proxy ensuring security and additional performances to your project. Furthermore it support SSL certificates to enable HTTPS connections.

Production mode can be enabled command line by adding the `--production` (or `--prod`) flag 

To ensure the option is always activated you can save it in a `.projectrc` file by adding:

`production: True`

### SSL Certificates

RAPyDo supports [Let's Encrypt](https://letsencrypt.org/) to automatically deploy SSL certificates. Once started your project in production mode you can request for a SSL certificate with the following command:

```
rapydo ssl
```

To let this command properly works please verify that:

1. you configured a correct hostname in your .projectrc
2. production mode is enabled
3. your stack is already started (with `rapydo start`), in particular the nginx container is expected to be running

If you want to issue a certificate without starting the container, you can use `rapydo ssl --volatile`

Let's Encrypt certificates expire in 90 days, you can renew them by executing again the `rapydo ssl` command.

To make sure your certificate is always up-to-date you can setup a cronjob to automatize the certificate renew. You can configure crontab to perform this work for you.

Crontab have some limitations due to the simplified environment used to execute commands, to overcome that limitations you have to:

1. provide absolute path to your RAPyDo executable (`/usr/local/bin/rapydo` in most cases)
2. you will haven't access to the command output. If you desire, your can redirect the output on a file

The following crontab entry is able to renew the SSL certificate every Monday at 00:00 AM

```
0 0 * * 1 cd /your/project/path && /usr/local/bin/rapydo ssl > /your/project/path/data/logs/ssl.log 2>&1 
```



## Upgrade to a new version

Let's assume that you already have a working deployment of your project (let's name it YOUR_PROJECT) based on version X and you want to upgrade to the new version Y.

**1 - Stop the current stack**

`rapydo remove`

**2 - Switch your git branch to the new version X**

`git fetch && git checkout X`

**3 - Upgrade your RAPyDo controller**

`rapydo install `

By default, the install command will understand by itself which version is required by YOUR_PROJECT. Otherwise you can specify a specific version with `rapydo install version`

**4 - Reinitialize your project**

`rapydo init`

This step will verify your submodules and will switch them to the correct branch if required.

**5 - Update your submodules**

`rapydo update`

This step is usually not needed when submodules are switched to a new version, but it is useful in any other case. You can safely add to your swiss-knife box and execute it in any case.

To skip updates of your main branch and only update your submodules: `rapydo update -i main`

**6 - Pull updated base images**

You can download updated base images with the `rapydo pull` command. This command will automatically fetch these images from the Docker Hub

**7 - Build project images (optional)**

If your project extends the base images you can build them with the `rapydo build`. This step is always optional, custom images will be automatically built when the stack will be executed if not previously built.

**8 - Upgrade completed**

Your upgrade procedure is now completed, you are able to start your stack with `rapydo start`



## Known issues post-upgrade

### PostgreSQL fails to start with RAPyDo 2.2

RAPyDo 2.2 upgraded the PostgreSQL version from 13.4 to 14.1. Databases created with psq13 are not compatible with psq14 and your container will fail to start with the following error:

  > FATAL:  database files are incompatible with server
  >
  > DETAIL:  The data directory was initialized by PostgreSQL version 14, which is not compatible with this version 14.1.

Based on the [Official Upgrading Guide](https://www.postgresql.org/docs/11/upgrading.html), to upgrade your database you have to restore a dump of your DB on the new engine. To ease the process RAPyDo includes a utility script (`version_upgrade`) bundled with the PostgreSQL build that automatically performs most of the steps needed to upgrade.  If your database is only used to store sessions you can simply destroy the database volume and re-initialize it. Otherwise you can perform a backup of your current database and then use the `version_upgrade` utility to restore the backup on the new version.

1. Make sure that you have a backup of your database, it will be used to create your new DB
   1. Backups are stored in `data/backup/postgres/`. Make sure to have a very recent backup of your db. You can also list your backups with `rapydo restore postgres`
   2. If you already have a backup go to step 2, otherwise you have to downgrade your project  (to PostgreSQL 13) to make a backup (you cannot create a backup of your data by using PostgreSQL 14 because the server is unable to start)
   3. Once downgraded your project to PostgreSQL 13 make a backup with `rapydo backup postgres`
   4. Upgrade again your project to PostgreSQL 14
2. You have a recent backup that will be restored to upgrade your database to PostgreSQL 14
   1. Since the PostgreSQL container crashes at startup due to incompatibility between your data and the new engine, you have to start a volatile container to prevent server execution: `rapydo run --debug postgres`
   2. execute `version_upgrade` to start the upgrade process and follow the instructions. The command will list your available backup files, to start the upgrade on a specific backup execute `version_upgrade backupfilename.sql.gz`
3. Exit the volatile container and start your server

The same issue already happened with RAPyDo 0.9 with the upgrade of PostgreSQL from 12 to 13 and the same will happen again when we will upgrade from version 14 to version 15



### Neo4j fails to start with RAPyDo 2.0

RAPyDo 2.0 upgraded neo4j from 4.2 to 4.3 and according to [neo4j official documentation](https://neo4j.com/docs/operations-manual/4.3/upgrade/deployment-upgrading/) a database conversion is needed.

To execute the upgrade:

1) start neo4j in upgrade mode `rapydo -e NEO4J_ALLOW_UPGRADE=True -s neo4j start `

2) your database is now upgraded, you can execute your stack as usual





