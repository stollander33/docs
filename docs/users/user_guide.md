***Work in progress...***

# Index of contents

*to be defined...*

[TOC]



# Main features

## Project Configuration

Each project has three level of configuration:

- default configuration (defined by RAPyDo it-self)
- project configuration (defined by a developer in `projects/YOUR_PROJECT/project_configuration.yml`)
- instance configuration (defined by the user in a `.projectrc` file)

A .projectrc file (created in the root of the project) can customize most of options by ovverriding configurations at project and RAPyDo levels

Here a typical `.projectrc` file:



`project: your_project`

`stack: your_custom_stack`

`hostname: localhost`

`# production: True`

`project_configuration:`
  `variables:`
    `env:`
      `SMTP_ADMIN: your@email.org`

​      `other variabiles...`



## Stacks

In every project there is a set configurations that you would like to use to switch one from another quickly. Even if not the case there should be at least two modes: 

1. `development` for working locally (verbose logs, auto reload and no reverse proxy)
2. `production` for deploying the same code in production (less logs, `uwsgi` and a reverse proxy, all set to use `SSL` for free with Letsencrypt).

Stacks are just implemented with a `YAML` file following the usual `compose` conventions. You can create as many custom stacks as you need.

The final configuration you will use in every RAPyDo command is based on:

- A RAPyDo base configuration, gathering the best practices on the most commonly used services
- A common custom configuration across different modes in your project (in `projects/YOUR_PROJECT/confs/commons.yml`)
- A custom file dedicated to a single mode  (in `projects/YOUR_PROJECT/confs/YOUR_STACK.yml`)

Default stacks (development and production) are automatically enabled

## Services

RAPyDo is of course containers oriented. This means that services can be easily added to be tested locally. We tested the most commons in our projects thus they are already integrated and all set:

- Postgres
- MariaDB
- Neo4J
- MongoDB
- Redis
- RabbitMQ
- Celery
- Pure-FTP
- Angular
- Pushpin

More services can be added as long as you can provide a container image (official or not) for it; so basically always!

In a stack (e.g. in production) you may choose to switch to external existing services by simply overriding the SERVICE configuration (HOST, PORT, etc), without changing your endpoint code.

## Swagger

The [OpenAPI standard]() has helped us many times to show to clients the experience of HTTP API services without even using a frontend. This is why every endpoint defined with RAPyDo is fully OpenAPI-compliant and included in a Swagger specification file available at the /api/specs endpoint. NOTE: we are changing the way endpoints are defined within RAPyDo. This will affect how we will make use of Swagger and OpenAPI, but in every case the final output will continue to be fully fully OpenAPI-compliant.

## Containers builds

All core RAPyDo images are automatically built and pushed on the Docker Hub. You can download all required images by using the `rapydo pull` command. A project can extend a base image (e.g. to intall additional libraries required for custom endpoints). Custom images can be build by using the `rapydo build` command.

## Interfaces

A set of interfaces can be launched as containers to help with many services:

- flower (for celery)
- swaggerui
- adminer for SQL servers
- mongo express

All interfaces can be executed by using the `rapydo interfaces` command

## Multi projects

The same repository can host different projects. This action came handy quite a lot of times when a similar project has to be maintaned in the same repo by the same people.

If the project is only one (the usual situation) it is used as default.

To switch projects you can use the `--project` command line parameter or set it inside the `.projectrc` file.

## Production mode

In production mode an additional container based on [NGINX](https://www.nginx.com/) is added to your stack. NGINX is a reverse proxy ensuring security and additional performances to your project. Furthermore it support SSL certificates to enable HTTPS connections.

Production mode can be enabled command line by adding the `--production` (or `--prod`) flag 

To ensure the option is always activated you can save it in a `.projectrc` file by adding:

`production: True`

## SSL Certificates

RAPyDo supports [Let's Encrypt](https://letsencrypt.org/) to automatically deploy SSL certificates. Once started your project in production mode you can request for a SSL certificate with the following command:

```
rapydo ssl-certificate
```

To let this command properly works please verify that:

1. you configured a correct hostname in your .projectrc
2. production mode is enabled
3. your stack is already started (with rapydo start), in particular the nginx container is expected to be running

Let's Encrypt certificates expire in 90 days, you can renew them by executing again the `rapydo ssl-certificate` command.

# Upgrade to a new version

We assume that you already have a working deployment of your project (let's name it YOUR_PROJECT) based on version X and you want to upgrade to the new version Y.

#### 1 - Stop the current stack

`rapydo remove`



#### 2 - Switch your git branch to the new version X

`git fetch && git checkout X`



#### 3 - Upgrade your RAPyDo controller

`rapydo install auto`

By setting the *auto* flag,  RAPyDo will understand by itself which version is required for YOUR_PROJECT.

If your  installed version is too old (this flag is only available from version 0.6.6), the *auto* flag could not be supported. In this case you can understand the required version by using: `rapydo version` and then upgrade the controller with the command: `rapydo install VERSION`.

Otherwise you can install the latest version with pip and then use the *auto* flag:

`pip3 install --upgrade rapydo-controller && rapydo install auto`



#### 4 - Reinitialize your project

`rapydo init`

This step will verify your submodules and will switch them to the correct branch if required.



#### 5 - Update your submodules

`rapydo update`

To skip updates of your main branch and only update your submodules: `rapydo update -i main`



#### 6 - Pull new base images

You can download updated base images with the `rapydo pull` command. This command will automatically fetch these images from the Docker Hub



#### 7 - Build project images (optional)

If your project extends the base images you can build them with the `rapydo build`. This step is always optional, if missing custom images will be automatically built when the stack will be execued



#### 8 - Upgrade completed

Your upgrade procedure is now completed, you are able to start your stack with `rapydo start`

[Back to the index](#index-of-contents)

# Tips and Tricks

## Differences between start and restart

The two commands (`rapydo start` and `rapydo restart`) can be quite confusing and it can be difficult to understand when to use one and when the other. The start command is based on the `docker-compose up` command and it is able to start the docker containers. The restart command is based on the `docker-compose restart` command and it is able to restart the main service of a container. Furthermore the start command is able to re-create the container when docker definition changes (e.g. an environment variable changed, a volume is added, porting mapping is changed, docker image is rebuilt, etc).

Use the start or the restart command depends from the kind of changes that you need to deploy. If the changes are at docker level, then you have to use the start command. If the changes are at software and service level, you have to use the restart command.

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



# Known issues post-upgrade

## Errors during submit of celery taks in  RAPyDo 0.7.3

Backend fails to submit any celery task and raises the following error:

`[PreconditionFailed] Queue.declare: (406) PRECONDITION_FAILED - inequivalent arg 'x-max-priority' for queue 'celery' in vhost '/': received the value '10' of type 'signedint' but current is none`

RAPyDo 0.7.3 enabled priority management in celery and this requires a specific configuration in the celery queue in RabbitMQ that needs to be recreated by executing the following steps:

1. remove the rabbitmq container with `rapydo -s rabbit remove`

2. remove the rabbitmq volume with `docker volume rm $YOURPROJECT_rabbitdata`

3. start again the rabbitmq container with `rapydo -s rabbit start`

   

## Networks need to be recreated in RAPyDo 0.7.2+

Due to network constraints in some virtual environment, RAPyDo 0.7.2 implemented an option to change the MTU used to create networks (NETWORK_MTU, defaulted to 1500). This new option changed the network definition even if no NETWORK_MTU is specified and services fail to start with the following message:

`Network "xyz_net" needs to be recreated - option "com.docker.network.driver.mtu" has changed.`
`Remove previously created networks and try again (you can use rapydo remove --networks or docker system prune)`

Can you fix this error by executing on of the suggested commands:

- `rapydo remove --networks` (from rapydo 0.7.3+)
- `rapydo clean` (up to rapydo 0.7.2)
- `docker system prune` (this will also remove stopped containers and untagged images)

As an alternative option you can remove the network(s) by hand with `docker network rm` 



## PostgreSQL fails to start in RAPyDo 0.6.7

RAPyDo 0.6.7 upgraded the PostgreSQL version from 10.7 to 11.4. Databases created with psq10 are not compatible with psq11 and your container will fail to start with the following error:

  > FATAL:  database files are incompatible with server
  >
  > DETAIL:  The data directory was initialized by PostgreSQL version 10, which is not compatible with this version 11.4.

  

  If your database is only used to store sessions you can simply destroy the database volume and re-initialize it

1. stop the stack:  `rapydo remove`
2. delete the sql data volume: `docker volume rm YOUR_PROJECT_sqldata`
3. start the stack: `rapydo start`
4. enter your backend container: `rapydo shell backend`
5. re-initialize the database: `restapi init`
6. exit the backend container: `exit`

​    If you cannot lose your data, please refer to the [Official Upgrading Guide](https://www.postgresql.org/docs/11/upgrading.html)

## PostgreSQL fails to start in RAPyDo 0.7.1

RAPyDo 0.7.1 upgraded the PostgreSQL version from 11.5 to 12.1. Databases created with psq11 are not compatible with psq12 and your container will fail to start. Please follow the section above for all possible fixes.

## Celery/backend fail to start in RAPyDo 0.7.1

RAPyDo 0.7.1 introduced explicit password setting for RabbitMQ default credentials and, if missing, you will be asked to set RABBITMQ_USER and RABBITMQ_PASSWORD variables in your .projectrc file. You can set whatever you want and these variables will be used to create the default admin user in rabbit. But if your Rabbit instance has been previously created an admin user is already created with default guest/guest credentials. Celery and backend will be unable to connect to Rabbit due to a credentials mismatch. To fix this issue you have three options:

1. set RABBITMQ_USER: guest and RABBITMQ_PASSWORD: guest (unsecure)
2. destroy the rabbit volume to inject a new admin user (`rapydo remove && docker volume rm YOUR_PROJECT_rabbit && rapydo start `). You will lose all your Rabbit (this is irrilevant, except if you are using Rabbit for other purpose  than RAPyDo)
3. enter the rabbit container and create a new user / change the user password to match the settings

## ssl-certificate command fails in RAPyDo 0.6.7

If the ssl-certificate command fails with the following error:

`Register account Error: {"type":"urn:acme:error:badNonce","detail":"JWS has no anti-replay nonce","status": 400}` you are running an obsolete letsencrypt version and you have to upgrade it then retry again. To upgrade your version you can adopt one of the following fixes:

1. upgrade your project to RAPyDo version 0.7.0
2. if you are using RAPyDo 0.6.7 you can rebuild your proxy (rapydo -s proxy build -rf && rapydo start)
3. You can upgrade letsencrypt by yourself by executing the following command into the proxy container: `/acme/acme.sh --upgrade --home /acme`

[Back to the index](#index-of-contents)