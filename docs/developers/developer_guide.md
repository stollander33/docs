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
         * [Logs](#logs)
         * [Security Events Logs](#security-events-logs)
         * [Asynchronous tasks](#asynchronous-tasks)
         * [Unit tests](#unit-tests)
         * [Frontend framework](#frontend-framework)
      * [Upgrade to a new version](#upgrade-to-a-new-version)

<!-- Added by: mdantonio, at: mar 1 mar 2022, 19:10:06, CET -->

<!--te-->

# Developer Guide

## Create a new RAPyDo-based project

Start by installing requisites and the rapydo-controller (as for the [User guide](docs/users/quick_start_users.md))

1. Make sure you meet the pre-requisites on your machine:

   - Python3.8+ (and `pip`) 
   - Docker 20+ with Compose v2
   - Git

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
- Redis
- Celery
- Celery-beat

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

RAPyDo supports several backend database to store authentication information: relation databases via sqlalchemy (postgres, mysql, mariadb), neo4j, mongodb. You can set the preferred database by setting the `AUTH_SERVICE` variable. You can also disable the authentication layer at all by setting `AUTH_SERVICE=NO_AUTHENTICATION`.

A number of flags allow to enhance the security layer, included enabling 2-Factor authentication based on TOTP.

Here a list of variabiles that can be configured in `project_configuration.yaml` or in `.projectrc`:

- AUTH_VERIFY_PASSWORD_STRENGTH
  - Default: 1 (enabled)
  - Values: 0 or 1
  - Enable verification of password strenght (i.e. minimum length, check that the password can't match the previous and that cannot contains name, surname or email address)
- AUTH_MIN_PASSWORD_LENGTH
  - Default: 8
  - Values: int
  - Set the minimum password length that can be accepted
- AUTH_FORCE_FIRST_PASSWORD_CHANGE
  - Default: 0 (disabled)
  - Values: 0 or 1
  - Force the change of the first password set by administrators
- AUTH_MAX_PASSWORD_VALIDITY
  - Default 0 (disabled)
  - Values: int
  - Validity period in days after which the password will expired and the user will be forced to change it
- AUTH_DISABLE_UNUSED_CREDENTIALS_AFTER
  - Default 0 (disabled)
  - Values: 0 or 1
  - Automatically disabled inactive users and the given perod in days
- AUTH_MAX_LOGIN_ATTEMPTS
  - Default 8
  - Values: int
  - A numer of failed logins greater than the configured value will temporary block the credentials and prevent any further attempt. The ban period is configured through the AUTH_LOGIN_BAN_TIME variable
- AUTH_LOGIN_BAN_TIME
  - Default 3600
  - Values: int
  - Durations in seconds of the credentials ban after failed logins
- AUTH_SECOND_FACTOR_AUTHENTICATION
  - Default 0 (disabled)
  - Values: 0 or 1
  - Enabled Two Factor Authentication based on TOTP (e.g. Google Authenticator)
- AUTH_JWT_TOKEN_TTL
  - Default 2592000 (1 month in seconds)
  - Values: int
  - JWT Tokens duration in seconds
- AUTH_TOKEN_IP_GRACE_PERIOD:
  - Default 1800
  - Values: int
  - Amount of seconds before starting to evaluate IP address on token validation (use of token from a IP address different by the address used to generated the token is forbidden after the grace period)

### REST classes

RAPyDO is Object Oriented (based to the `Flask-Restful` and `flask-apispec` packages): each endpoint is mapped to a class and automatically configured. You can simply create a new python file in projects/YOUR_PROJECT/backend/endpoinits to define your class-endpoint and it will be automatically added to the project. You can add new endpoints by using the templating command: `rapydo add endpoint endpoint-name`

A REST class extends `EndpointResource` from `restapi.rest.definition`, defines methods one or more methods `get`, `post`, `put`, `patch` or `deleted` decorated by the `@decorators.endpoint` decorator used to describe path, description  and responses. This decorator is mostly based on apispec. Input and output can be specified respectively by the `@decorators.use_kwargs`(mostly based on webargs) and `@decorators.marshal_with` (mostly based on Marshmallow) decorators.

Here an example of a simple REST class:

```python
from restapi import decorators
from restapi.rest.definition import EndpointResource, Response

class MyFirstRESTClass(EndpointResource):

    # @decorators.auth.require()
    # @decorators.use_kwargs(MyWebargsInputSchema)
    # @decorators.marshal_with(MyMarshalOutputSchema)
    @decorators.endpoint(
        path="/api/myendpoint",
        summary="My first endpoint",
        description="This is my first endpoint, it simply reply with a Hello World",
        responses={
            200: "Hello World is replied",
        },
    )
    def get(self) -> Response:
        self.response("Hello World")

```





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
- `/api/groups/users`



### Logs

By default the backend embeds a wrapper of loguru that can be used by importing from the restapi.utilities package:

```
from restapi.utilities.logs import log
...
log.debug("My message")
log.info("My message")
log.warning("My message")
log.error("My message")
log.critical("My message")
```

Logs visibility is controlled by a global variable (LOG_LEVEL) that can be modified at project_configuration and .projectrc level (default value is DEBUG).

By default all the logs with severity equal or greater then the warning level are saved to the data/logs/backend-server.log file. This file is automatically rotated every week and retained for 6 months (controller by the LOG_RETENTION variable).

the log.exception function is not printed in standard output but only stored on the backend-server.log with a detailed error stack for debugging purpose.

### Security Events Logs

As an additional logging system, from every endpoint it possible to store information on the data/logs/security-events.log. As for the general log, also the security events log is automatically rotated every week and retained for 6 months (controller by the LOG_RETENTION variable).

To store onto the security events logs the endpoints the use the log_event utility:

```
myobj = db.myModel(kwargs).save()
self.log_event(self.events.create, myobj, kwargs)
```

Supported events type (first parameter) are:

- access
- create
- modify
- delete
- login
- logout
- failed_login
- refused_login
- activation
- login_unlock
- change_password
- reset_password_request
- password_expired
- server_startup

The second parameter is the event target (usually the object created/modified/delete) while the third is the payload used for the event. Both the parameters are optional.

The log is automatically extended with date, IP, user (in case of authenticated endpoint), object type and object id/uuid

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