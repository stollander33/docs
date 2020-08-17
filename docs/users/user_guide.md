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
      * [Tips and Tricks](#tips-and-tricks)
         * [Differences between start and restart](#differences-between-start-and-restart)
         * [Automatic certificate renew by using crontab](#automatic-certificate-renew-by-using-crontab)
      * [Known issues post-upgrade](#known-issues-post-upgrade)
         * [Neo4j fails to start in RAPyDo 0.7.4](#neo4j-fails-to-start-in-rapydo-074)
         * [Errors when submitting celery tasks in RAPyDo 0.7.3](#errors-when-submitting-celery-tasks-in-rapydo-073)
         * [Networks need to be recreated in RAPyDo 0.7.2 ](#networks-need-to-be-recreated-in-rapydo-072)
         * [PostgreSQL fails to start in RAPyDo 0.7.1](#postgresql-fails-to-start-in-rapydo-071)
         * [Celery/backend fail to start in RAPyDo 0.7.1](#celerybackend-fail-to-start-in-rapydo-071)
         * [ssl-certificate command fails in RAPyDo 0.6.7](#ssl-certificate-command-fails-in-rapydo-067)

<!-- Added by: mdantonio, at: lun 17 ago 2020, 14:21:23, CEST -->

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

Stacks are just implemented as `YAML` files based on the `docker-compose` syntax. You can create as many custom stacks as you need.

The final configuration you will use in every RAPyDo command is based on:

- A RAPyDo base configuration, gathering the best practices on the most commonly used services
- A common custom configuration across different modes in your project (in `projects/YOUR_PROJECT/confs/commons.yml`)
- A custom file dedicated to a single mode  (in `projects/YOUR_PROJECT/confs/YOUR_STACK.yml`)

Default stacks (development and production) are automatically enabled

You can enable production stack by passing --production (or --prod) option to any RAPyDo command or by setting production: True in your .projectrc. You can enable any other custom stack by passing --stack STACK_NAME to any RAPyDo command or by setting stack: STACK_name in your .projectrc.

### Services

RAPyDo is of course containers oriented. This means that services can be easily added to be tested locally. We tested the most commons in our projects thus they are already integrated:

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
- adminer for SQL servers
- mongo express

All interfaces can be executed by using the `rapydo interfaces` command.

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

Let's Encrypt certificates expire in 90 days, you can renew them by executing again the `rapydo ssl` command or by setting a cronjob like this:

`0 0 * * 2 cd /prj/path && COMPOSE_INTERACTIVE_NO_CLI=1 /usr/local/bin/rapydo ssl --no-tty > /logs/cron.log`

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

**6 - Pull new base images**

You can download updated base images with the `rapydo pull` command. This command will automatically fetch these images from the Docker Hub

**7 - Build project images (optional)**

If your project extends the base images you can build them with the `rapydo build`. This step is always optional, custom images will be automatically built when the stack will be executed if not previously built.

**8 - Upgrade completed**

Your upgrade procedure is now completed, you are able to start your stack with `rapydo start`

## Tips and Tricks

### Differences between start and restart

The two commands (`rapydo start` and `rapydo restart`) can be quite confusing and it can be difficult to understand when to use one and when the other. The start command is based on the `docker-compose up` command and it is able to start the docker containers. The restart command is based on the `docker-compose restart` command and it is able to restart the main service of a container. Furthermore the start command is able to re-create the container when the docker definition changes (e.g. an environment variable changed, a volume is added, porting mapping is changed, docker image is rebuilt, etc).

Use the start or the restart command depends from the kind of changes that you need to deploy. If the changes are at docker level, then you have to use the start command. If the changes are at software and service level, you have to use the restart command.

### Automatic certificate renew by using crontab

To make sure your certificate is always up-to-date you can setup a cronjob to automatize the certificate renew. You can configure crontab to perform this work for you.

Crontab have some limitations due to the simplified environment used to execute commands, to overcome that limitations you have to:

1. provide absolute path to your RAPyDo executable (`/usr/local/bin/rapydo` in most cases)
2. set `COMPOSE_INTERACTIVE_NO_CLI=1` to prevent Compose to use the Docker CLI for interactive `run` and `exec`operations.
3. enable `--no-tty` flag to disable pseudo-tty allocation (by default docker-compose allocates a TTY, not available from crontab)
4. you will haven't access to the command output. If you desire, your can redirect the output on a file

The following crontab entry is able to renew the SSL certificate every Monday at 00:00 AM

```
0 0 * * 1 cd /your/project/path && COMPOSE_INTERACTIVE_NO_CLI=1 /usr/local/bin/rapydo ssl --no-tty > /your/project/data/logs/cron.log 2>&1 
```



## Known issues post-upgrade

### Neo4j fails to start in RAPyDo 0.7.4

RAPyDo 0.7.4 changed the uid/gid of the neo4j user in the corresponding container. Previosuly created database can have wrong permissions and fail to start due to permission denied. You can fix it by changing the ownership of the data folder with your own user (the user you use on the host machine).

If you deployed neo4j from a local folder (e.g. `data/graphdata` ) you can simply change the ownership:

`chown -R your_user:your_group data/graphdata`

If you deployed neo4j from a docker volume (this is the default) you have to perform this operation from inside the container. You can execute a volatile neo4j container and change the ownership of the data folder:

`rapydo volatile neo4j`

`chown -R neo4j:neo4j /data`

### Errors when submitting celery tasks in RAPyDo 0.7.3

Backend fails to submit any celery task and raises the following error:

`[PreconditionFailed] Queue.declare: (406) PRECONDITION_FAILED - inequivalent arg 'x-max-priority' for queue 'celery' in vhost '/': received the value '10' of type 'signedint' but current is none`

RAPyDo 0.7.3 enabled priority management in celery and this requires a specific configuration in the celery queue in RabbitMQ that needs to be recreated by executing the following steps:

1. remove the rabbitmq container with `rapydo -s rabbit remove`

2. remove the rabbitmq volume with `docker volume rm $YOURPROJECT_rabbitdata`

3. start again the rabbitmq container with `rapydo -s rabbit start`

   

### Networks need to be recreated in RAPyDo 0.7.2+

Due to network constraints in some virtual environment, RAPyDo 0.7.2 implemented an option to change the MTU used to create networks (NETWORK_MTU, defaulted to 1500). This new option changed the network definition even if no NETWORK_MTU is specified and services fail to start with the following message:

`Network "xyz_net" needs to be recreated - option "com.docker.network.driver.mtu" has changed.`
`Remove previously created networks and try again (you can use rapydo remove --networks or docker system prune)`

Can you fix this error by executing on of the suggested commands:

- `rapydo remove --networks` (from rapydo 0.7.3+)
- `rapydo clean` (up to rapydo 0.7.2)
- `docker system prune` (this will also remove stopped containers and untagged images)

As an alternative option you can remove the network(s) by hand with `docker network rm` 



### PostgreSQL fails to start in RAPyDo 0.7.1

RAPyDo 0.7.1 upgraded the PostgreSQL version from 11.5 to 12.1. Databases created with psq11 are not compatible with psq12 and your container will fail to start with the following error:

  > FATAL:  database files are incompatible with server
  >
  > DETAIL:  The data directory was initialized by PostgreSQL version 11, which is not compatible with this version 12.1.

​    If your database is only used to store sessions you can simply destroy the database volume and re-initialize it

1. stop the stack:  `rapydo remove`
2. delete the sql data volume: `docker volume rm YOUR_PROJECT_sqldata`
3. start the stack: `rapydo start`
4. enter your backend container: `rapydo shell backend`
5. re-initialize the database: `restapi init`
6. exit the backend container: `exit`

​    If you cannot lose your data, please refer to the [Official Upgrading Guide](https://www.postgresql.org/docs/11/upgrading.html)

The same issue already happened with RAPyDo 0.6.7 with the upgrade of PostgreSQL from 10.7 to 11.4 and the same will happen again when we will upgrade from version 12.x to version 13.x

### 

### Celery/backend fail to start in RAPyDo 0.7.1

RAPyDo 0.7.1 introduced explicit password setting for RabbitMQ default credentials and, if missing, you will be asked to set RABBITMQ_USER and RABBITMQ_PASSWORD variables in your .projectrc file. You can set whatever you want and these variables will be used to create the default admin user in rabbit. But if your Rabbit instance has been previously created an admin user is already created with default guest/guest credentials. Celery and backend will be unable to connect to Rabbit due to a credentials mismatch. To fix this issue you have three options:

1. set RABBITMQ_USER: guest and RABBITMQ_PASSWORD: guest (unsecure)
2. destroy the rabbit volume to inject a new admin user (`rapydo remove && docker volume rm YOUR_PROJECT_rabbit && rapydo start `). You will lose all your Rabbit (this is irrilevant, except if you are using Rabbit for other purpose  than RAPyDo)
3. enter the rabbit container and create a new user / change the user password to match the settings

### ssl-certificate command fails in RAPyDo 0.6.7

If the ssl-certificate command fails with the following error:

`Register account Error: {"type":"urn:acme:error:badNonce","detail":"JWS has no anti-replay nonce","status": 400}` you are running an obsolete letsencrypt version and you have to upgrade it then retry again. To upgrade your version you can adopt one of the following fixes:

1. upgrade your project to RAPyDo version 0.7.0
2. if you are using RAPyDo 0.6.7 you can rebuild your proxy (rapydo -s proxy build -rf && rapydo start)
3. You can upgrade letsencrypt by yourself by executing the following command into the proxy container: `/acme/acme.sh --upgrade --home /acme`
