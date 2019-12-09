***Work in progress...***

# Index of contents

[TOC]

# Create a new RAPyDo-based project

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



# Enable a service

You can list active services with `rapydo list --services`

Active services are a combination of services enabled by configuring the `ACTIVATE_SERVICENAME` environment variable in `projects/$PROJECT_NAME/project_configuration.yaml` and services automatically actived due to dependency rules

For example by configuring `ACTIVATE_CELERY: 1` in `project_configuration.yaml` both Celery and RabbitMQ will be enabled. Celery is configured by default to use RabbitMQ as both broker and backend. If you prefer to configure MongoDB as Celery backend (recommended choice) you can add the following configuration in `project_configuration.yaml`:

    ACTIVATE_MONGODB: 1
    CELERY_BACKEND: MONGODB
    CELERY_BACKEND_HOST: mongo.dockerized.io
    CELERY_BACKEND_PORT: 27017


# Implement new endpoints

...



# Add a new service

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
