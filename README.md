# RAPyDo framework

A containers-based framework to develop your HTTP-API with Flask on Python3 with optional web interface written with Angular. RAPyDO also includes services like relational databases (PostgreSQL, MariaDB), graph database (Neo4j), no-sql database (MongoDB), Celery for asynchronous operations (based on RabbitMQ, also included in the box) and Pushpin for WebSockets connections, Redis and much more

The name is an acronymous for:

- `R`ESTful `A`PI
- on a `Py`thon server
- based on a `Do`cker



## Why to use RAPyDo?

Along our journey into efficient HTTP API middleware towards distributed services we had to reach some level of stability within our environment.

Having the same tasks to be completed over and over we decided to develop RAPyDo to have a common base to all our RESTful HTTP API oriented projects. 
What we have so far is what helped us in speeding up setup and development process while keep track of all the solutions we found in all the problems we encountered in at least 5 years of italian/european projects experience.

RAPyDo can be used from both the final user and the developer.

The user usually interact with an already-existing RAPyDo-based project to deploy, monitor and manage one or more project installations. The developer creates a  RAPyDo-based project , customize the stack, implements the endpoints.

Do you already have a RAPyDo-based project? You have to deploy it on a new server? Do you want to test an installation? Let's start with the [User Quick Start](docs/users/quick_start_users.md) and the [User Guide](docs/users/user_guide.md)

Do you want to create a new RAPyDo-based project? You have to implement a new endpoint? You need to configure a service? Let's start with the [Developer Guide](docs/developers/developer_guide.md)



# Table of Contents

   * [User Guide](docs/users/user_guide.md#user-guide)
      * [Project Configuration](docs/users/user_guide.md#project-configuration)
         * [Stacks](docs/users/user_guide.md#stacks)
         * [Services](docs/users/user_guide.md#services)
         * [Swagger](docs/users/user_guide.md#swagger)
         * [Containers builds](docs/users/user_guide.md#containers-builds)
         * [Interfaces](docs/users/user_guide.md#interfaces)
         * [Multi projects](docs/users/user_guide.md#multi-projects)
         * [Production mode](docs/users/user_guide.md#production-mode)
         * [SSL Certificates](docs/users/user_guide.md#ssl-certificates)
      * [Upgrade to a new version](docs/users/user_guide.md#upgrade-to-a-new-version)
      * [Tips and Tricks](docs/users/user_guide.md#tips-and-tricks)
         * [Differences between start and restart](docs/users/user_guide.md#differences-between-start-and-restart)
         * [Automatic certificate renew by using crontab](docs/users/user_guide.md#automatic-certificate-renew-by-using-crontab)
      * [Known issues post-upgrade](docs/users/user_guide.md#known-issues-post-upgrade)
         * [Errors when submitting celery taks in RAPyDo 0.7.3](docs/users/user_guide.md#errors-when-submitting-celery-taks-in-rapydo-073)
         * [Networks need to be recreated in RAPyDo 0.7.2 ](docs/users/user_guide.md#networks-need-to-be-recreated-in-rapydo-072)
         * [PostgreSQL fails to start in RAPyDo 0.7.1](docs/users/user_guide.md#postgresql-fails-to-start-in-rapydo-071)
         * [Celery/backend fail to start in RAPyDo 0.7.1](docs/users/user_guide.md#celerybackend-fail-to-start-in-rapydo-071)
         * [ssl-certificate command fails in RAPyDo 0.6.7](docs/users/user_guide.md#ssl-certificate-command-fails-in-rapydo-067)

   * [Developer Guide](docs/developers/developer_guide.md#developer-guide)
      * [Create a new RAPyDo-based project](docs/developers/developer_guide.md#create-a-new-rapydo-based-project)
      * [Services](docs/developers/developer_guide.md#services)
         * [Enable a service](docs/developers/developer_guide.md#enable-a-service)
         * [Enable Rabbit Management Plugin](docs/developers/developer_guide.md#enable-rabbit-management-plugin)
         * [Add a new service](docs/developers/developer_guide.md#add-a-new-service)
      * [Backend development](docs/developers/developer_guide.md#backend-development)
         * [Security](docs/developers/developer_guide.md#security)
         * [REST classes](docs/developers/developer_guide.md#rest-classes)
         * [Base endpoints](docs/developers/developer_guide.md#base-endpoints)
         * [Services injections](docs/developers/developer_guide.md#services-injections)
         * [ORM](docs/developers/developer_guide.md#orm)
         * [Asynchronous tasks](docs/developers/developer_guide.md#asynchronous-tasks)
         * [Unit tests](docs/developers/developer_guide.md#unit-tests)
         * [Frontend framework](docs/developers/developer_guide.md#frontend-framework)
      * [Upgrade to a new version](docs/developers/developer_guide.md#upgrade-to-a-new-version)

