*Developer Guide - Table of Contents*

<!--ts-->
   * [Developer Guide](#developer-guide)
      * [Create a new RAPyDo-based project](#create-a-new-rapydo-based-project)
      * [Services](#services)
         * [Enable a service](#enable-a-service)
         * [Enable Rabbit Management Plugin](#enable-rabbit-management-plugin)
         * [Add a new service](#add-a-new-service)
      * [Backend development](#backend-development)
         * [Security](#security)
         * [REST classes](#rest-classes)
         * [Base endpoints](#base-endpoints)
         * [Services injections](#services-injections)
         * [ORM](#orm)
         * [Asynchronous tasks](#asynchronous-tasks)
         * [Unit tests](#unit-tests)
         * [Frontend framework](#frontend-framework)
      * [Upgrade to a new version](#upgrade-to-a-new-version)

<!-- Added by: mdantonio, at: mar 12 mag 2020, 10:46:18, CEST -->

<!--te-->

# Developer Guide

## Create a new RAPyDo-based project

Start by installing requisites and the rapydo-controller (as for the [User guide](docs/users/quick_start_users.md))

1. Make sure you meet the pre-requisites on your machine:

   - Python3.5+ (and `pip`) 
   - Git
   - Docker daemon/engine

2. install the rapydo controller

   Install the latest version: `pip3 install --upgrade rapydo-controller`

   Your project could require a different version, in this case you will be able to install the right version once configured your project

3. Create a new project by using

   `rapydo create template name`

   template is the base project to start from. Allowed values are: sql, neo, mongo, celerytest, irodstest, centos. name is name of your new project

   - sql: template project based on SQLAlchemy with PostgreSQL with Angular frontend enabled
   - neo: template project based on neo4j with Angular frontend enabled
   - mongo: template project based on MongoDB with Angular frontend enabled
   - celerytest: as for the sql template with Celery and Angular frontend enabled
   - irodstest: as for the sql template with iRODS and Angular frontend  enabled
   - centos: template project based on SQLAlchemy with PostgreSQL deployed on Centos (all other projects are based on Ubuntu) with Angular frontend enabled

4. A folder with the select name will be created. Follow instructions printed by rapydo create output, i.e.
   - cd your_project
   - git init
   - git remote add origin https://your_remote_git/your_project.git
   - rapydo init
5. Now you can edit your `projects/$PROJECT_NAME/project_configuration.yaml` to customize project title, description, the default user and database password.
6. The configuration is now complete, you can start your project by following the User guide or continue to customize by enabling more services or implement endpoints



## Services

### Enable a service

You can list active services with `rapydo list --services`

Active services are a combination of services enabled by configuring the `ACTIVATE_SERVICENAME`  variable in `projects/$PROJECT_NAME/project_configuration.yaml` or `.projectrc` and services automatically activated due to dependency rules.

By activating a service both backend  will  establish a connection by using one of the confgured extension:

- SQLAlchemy
- Neo4j
- MongoDB
- RabbitMQ
- Celery
- Celery-beat
- iRODS
- Pushpin

If you want to activate the service but NOT the connection from the backend you can add a `SERVICENAME_ENABLE_CONNECTOR=True` variable in `projects/$PROJECT_NAME/project_configuration.yaml` or `.projectrc` 

If you want to activate the connection from the backend but NOT a container for the service, you can configure the SERVICENAME_HOST with an external host.

To summarize:

Service container enabled + Backend connection enable -> ACTIVATE_SERVICENAME

Service container enabled + Backend connection disabled -> ACTIVATE_SERVICENAME && SERVICENAME_ENABLE_CONNECTOR

Service container disabled + Backend connection enable -> SERVICENAME_HOST

Service container disabled + Backend connection disabled -> nothing



### Enable Rabbit Management Plugin

Management plugin is disabled by default, you can activate by adding the following variabile to your project_configuration or .proectrc:

`RABBITMQ_ENABLE_MANAGEMENT_PLUGIN: 1`

You also have to add the following port mapping to your stack configuration (commons.yml to be shared with all stack or into a specific stack configuration):

  `rabbit:`
    `ports:`
      `- ${RABBITMQ_MANAGEMENT_PORT}:${RABBITMQ_MANAGEMENT_PORT}`

To enable access to queue you can also add:

​      `- ${RABBITMQ_PORT}:${RABBITMQ_PORT}`

### Add a new service

RAPyDo is designed to be easily extended with new services in addition to those already provided. For instance, let's suppose we want to add Apache NiFi. We have to add the new service in {custom}/confs/common.yml and activate it from the {custom}/project_configuration.yml

common.yml:

```
services:
  ...
  nifi:
    image: apache/nifi
    ports:
      - 8070:8080
    environment:
      ACTIVATE: ${ACTIVATE_NIFI}
```

and activate it in project_configuration.yml

```
variables:
  env:
    ...
    ACTIVATE_NIFI: 1
```

and `rapydo start` will do the rest



## Backend development

(warning: copy-pasted from an old documentation, to be revised!)

### Security

Any service available in RAPyDo as an ORM can be used as authentication for the system, you just need to switch the dedicated variable `AUTH_SERVICE`.

Oauth2 and 2 factor authentication is already integrated (through TOTP).
The nginx reverse proxy has been tested in many production system.

### REST classes

RAPyDO is Object Oriented (thanks to the `Flask-Restful` plugin): each endpoint is mapped to a class and automatically configured. The class associated to an endpoint is provided in the swagger configuration (and then removed from the public view). One method can be mapped to multiple endpoints paths (e.g. if you need some aliasing).

### Base endpoints

Helper endpoints are provided out of the box:

- `/api/status`
- `/api/specs`
- If you enable authentication:
- `/auth/login`
- `/auth/logout`
- `/auth/tokens`
- `/auth/profile`

### Services injections

The real first pain point we had to solve in our experience while working with Flask when containers were yet limited in their experience (at least a few years ago) was a dynamic injections of only the services configurated in the RAPyDO YAML files.

We created a set of defaults for each service. Each service can be activated with `${SERVICE}_ENABLE` or by adding the dependency on an already active service, and by leaving the defaults a container will be created for it and linked to the backend server/container and accessed by a simple `self.get_instance(my_service, **custom_args)` function call inside your endpoint code.

It doesn't matter what is your mode (internal with no credentials, or external with passwords), the code will remain the same!

NOTE: connections are kept globally in a pool to optimize the consumption of resources; you can force an instance to be recreated.

### ORM

Each service may be used as an ORM, if the equivalent python library exists. The base models and the custom models are pre loaded into the service objects at server startup.

This means that you can do into your code:

```python
mongo = self.get_instance('mongo')
mongo.MyModel(field1='yes', field2=False).save()
```

Where `MyModel` was defined inside your project custom `models/mongo.py` file.

For each service base models are provided to describe a User/Role approach to authentication.

### Asynchronous tasks

Celery tasks can be easily activated to be able to launch and control from any endpoint asynchronous tasks.
Tasks can be monitored directly from endpoints or from the `flower` UI.
Tasks can save progress, return status, send emails.

### Unit tests

`py.tests` is supported by default.
A unit test is a class in a separated `tests` folder, where you extend the existing base class from where you inherit methods to authenticate and handle tokens.



### Frontend framework

`Angular` is already integrated as base framework for the frontend part.
The base authentication (profile, change password, reset password, session lists, JWT tokens) endpoints are already tested inside the base TS code.

In debug mode the framework is served on nodejs/webpack while in production the static dist is built at startup time by the related container entrypoint.

NOTE: we are looking for `react` to be integrated as well!



## Upgrade to a new version

Let's say your project is based on version A and you want to upgrade it to version B.

1. edit your project_configuration.yml to change project.rapydo from A to B. Probably you also want to increase your project.version
2. install rapydo-controller version B (now that you changed the required RAPyDo version in project_configuration.yml you are able to upgrade your controller with `rapydo install auto`)
3. if you have custom docker builds change the base image from `rapydo/base-image:A`to `rapydo/base-image:B`
4. execute the upgrade of your local instance, as describe in [User Guide](docs/users/user_guide.md#upgrade-to-a-new-version)

Your project is now upgraded to version B