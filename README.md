# RAPyDo framework (v2.2)

A containers-based framework to develop your HTTP-API with Flask on Python with optional web interface written in Angular. RAPyDO also includes services like relational databases (PostgreSQL, MariaDB), graph database (Neo4j), NoSQL database (Redis, MongoDB), Celery for asynchronous operations, RabbitMQ as message broker and much more

The name is an acronym for:

- `R`ESTful `A`PI
- on a `Py`thon server
- based on a `Do`cker

RAPyDO is able to deploy your stack on both Swarm and Compose v2.

## Why to use RAPyDo?

Along our journey into efficient HTTP API middle-ware towards distributed services we had to reach some level of stability within our environments.

Having the same patterns to be implemented and managed over and over again, we decided to develop RAPyDo to have a common base to all our RESTful HTTP API oriented projects.
What we have so far is what helped us in speeding up setup and development process while keeping track of all the solutions we found in all the problems we encountered in at least 6 years of European projects experience.

RAPyDo can be used from both the final user and the developer.

The user usually interact with an already-existing RAPyDo-based project to deploy, monitor and manage one or more installations. The developer creates a  RAPyDo-based project , customize the stack, implements the endpoints.

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
      * [Known issues post-upgrade](docs/users/user_guide.md#known-issues-post-upgrade)
         * [PostgreSQL fails to start with RAPyDo 2.2](docs/users/user_guide.md#postgresql-fails-to-start-with-rapydo-22)
         * [Neo4j fails to start with RAPyDo 2.0](docs/users/user_guide.md#neo4j-fails-to-start-with-rapydo-20)

   * [Developer Guide](docs/developers/developer_guide.md#developer-guide)
      * [Create a new RAPyDo-based project](docs/developers/developer_guide.md#create-a-new-rapydo-based-project)
      * [Services](docs/developers/developer_guide.md#services)
         * [Enable a service](docs/developers/developer_guide.md#enable-a-service)
         * [Add a new service](docs/developers/developer_guide.md#add-a-new-service)
      * [Backend development](docs/developers/developer_guide.md#backend-development)
         * [Security](docs/developers/developer_guide.md#security)
         * [REST classes](docs/developers/developer_guide.md#rest-classes)
         * [Base endpoints](docs/developers/developer_guide.md#base-endpoints)
         * [Logs](docs/developers/developer_guide.md#logs)
         * [Security Events Logs](docs/developers/developer_guide.md#security-events-logs)
         * [Asynchronous tasks](docs/developers/developer_guide.md#asynchronous-tasks)
         * [Unit tests](docs/developers/developer_guide.md#unit-tests)
         * [Frontend framework](docs/developers/developer_guide.md#frontend-framework)
      * [Upgrade to a new version](docs/developers/developer_guide.md#upgrade-to-a-new-version)

