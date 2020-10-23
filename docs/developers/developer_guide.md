*Developer Guide - Table of Contents*

<!--ts-->
   * [Developer Guide](#developer-guide)
      * [Create a new RAPyDo-based project](#create-a-new-rapydo-based-project)
      * [Services](#services)
         * [Enable a service](#enable-a-service)
         * [Add a new service](#add-a-new-service)
      * [Backend development](#backend-development)
         * [Security](#security)
         * [REST classes](#rest-classes)
         * [Base endpoints](#base-endpoints)
         * [Asynchronous tasks](#asynchronous-tasks)
         * [Unit tests](#unit-tests)
         * [Frontend framework](#frontend-framework)
      * [Upgrade to a new version](#upgrade-to-a-new-version)

<!-- Added by: mdantonio, at: ven 23 ott 2020, 21:39:01, CEST -->

<!--te-->

# Developer Guide

## Create a new RAPyDo-based project

Start by installing requisites and the rapydo-controller (as for the [User guide](docs/users/quick_start_users.md))

1. Make sure you meet the pre-requisites on your machine:

   - Python3.6+ (and `pip`) 
   - Git
   - Docker daemon/engine

2. install the rapydo controller

   Install the latest version: `pip3 install --upgrade rapydo`

   Your project could require a different version, in this case you will be able to install the right version once configured your project

3. Create and enter a new directory

4. Create a new project by using

   `rapydo create name --auth YOUR_AUTH_SERVICE --frontend YOUR_FRONTEND ...other-options`  

5. Follow instructions printed by the rapydo create output, i.e.
   - git remote add origin https://your_remote_git/your_project.git
   - rapydo init
   - rapydo pull

6. Now you can edit your `projects/$PROJECT_NAME/project_configuration.yaml` to customize project title, description, enabled services and so on. Please note that you can enable services, ad other options, by setting up the rapydo create command. Use rapydo create --help to get the list of available options.

7. The configuration is now complete, you can start your project by following the User guide or further customize your project.



## Services

### Enable a service

You can list active services with `rapydo list services`

Active services are a combination of services enabled by configuring the `ACTIVATE_SERVICENAME`  variable in `projects/$PROJECT_NAME/project_configuration.yaml` or `.projectrc` and services automatically activated due to dependency rules. At creation time you can enable new services by adding the `--service` flag to the `rapydo create` command

By activating a service the backend server will establish a connection by using one of the configured extension:

- SQLAlchemy
- Neo4j
- MongoDB
- RabbitMQ
- Celery
- Celery-beat
- Pushpin

If you want to activate the service but NOT the connection from the backend you can add a `SERVICENAME_ENABLE_CONNECTOR=False` variable in `projects/$PROJECT_NAME/project_configuration.yaml` or `.projectrc` 

If you want to activate the connection from the backend but NOT a container for the service, you can configure the SERVICENAME_HOST with an external host.

To summarize:

Service container enabled + Backend connection enable -> ACTIVATE_SERVICENAME

Service container enabled + Backend connection disabled -> ACTIVATE_SERVICENAME && SERVICENAME_ENABLE_CONNECTOR

Service container disabled + Backend connection enable -> SERVICENAME_HOST

Service container disabled + Backend connection disabled -> nothing



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

### Security

Any service available in RAPyDo as an ORM can be used as authentication for the system, you just need to switch the dedicated variable `AUTH_SERVICE`. Additionally 2-Factor authentication can be enabled (base on TOTP).

### REST classes

RAPyDO is Object Oriented (based to the `Flask-Restful` and `flask-apispec` libraries): each endpoint is mapped to a class and automatically configured. You can simply create a new python file in projects/YOUR_PROJECT/backend/apis to define your class-endpoint and it will be automatically added to the project. You can add new endpoints by using the templating commando: `rapydo add endpoint endpoint-name`

### Base endpoints

Helper endpoints are provided out of the box:

- `/api/status`
- `/api/specs`
- `/auth/login`
- `/auth/logout`
- `/auth/tokens`
- `/auth/profile`
- `/auth/status`
- `/api/admin/users`
- `/api/admin/sessions`
- `/api/admin/stats`

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

In debug mode the framework is dynamically compiled and provided by ng serve while in production the static dist is built at startup time and served by a nginx reverse proxy.



## Upgrade to a new version

Let's say your project is based on version A and you want to upgrade it to version B.

1. edit your project_configuration.yml to change project.rapydo from A to B. Probably you also want to increase your project.version
2. install rapydo-controller version B (now that you changed the required RAPyDo version in project_configuration.yml you are able to upgrade your controller with `rapydo install`)
3. if you have custom docker builds change the base image from `rapydo/base-image:A`to `rapydo/base-image:B`
4. execute the upgrade of your local instance, as describe in [User Guide](docs/users/user_guide.md#upgrade-to-a-new-version)

Your project is now upgraded to version B