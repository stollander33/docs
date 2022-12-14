# RAPyDo framework (v2.4)

Un marco basado en contenedores para desarrollar su HTTP-API con Flask en Python con una interfaz web opcional escrita en Angular.

RAPyDO también incluye varias bases de datos (relacionales como PostgreSQL, MariaDB, gráficas como Neo4j, NoSQL como Redis), Celery para operaciones asíncronas, RabbitMQ como intermediario de mensajes y mucho más.

The name is an acronym for:

- `R`ESTful `A`PI
- on a `Py`thon server
- based on a `Do`cker

RAPyDO is able to deploy your stack on both Swarm and Compose v2.

## Why to use RAPyDo?

A lo largo de nuestro viaje hacia el middleware API HTTP eficiente hacia los servicios distribuidos, tuvimos que alcanzar cierto nivel de estabilidad dentro de nuestros entornos.

Teniendo los mismos patrones para ser implementados y administrados una y otra vez, decidimos desarrollar RAPyDo para tener una base común para todos nuestros proyectos orientados a RESTful HTTP API. Lo que tenemos hasta ahora es lo que nos ayudó a acelerar el proceso de configuración y desarrollo mientras realizamos un seguimiento de todas las soluciones que encontramos en todos los problemas que encontramos en al menos 6 años de experiencia en proyectos europeos.

RAPyDo puede ser utilizado tanto por el usuario final como por el desarrollador.

El usuario suele interactuar con un proyecto basado en RAPyDo ya existente para implementar, monitorear y administrar una o más instalaciones. El desarrollador crea un proyecto basado en RAPyDo, personaliza la pila e implementa los puntos finales.

¿Ya tienes un proyecto basado en RAPyDo? ¿Tienes que implementarlo en un nuevo servidor? ¿Quieres probar una instalación? Comencemos con [User Quick Start](docs/users/quick_start_users.md) and the [User Guide](docs/users/user_guide.md)


¿Quieres crear un nuevo proyecto basado en RAPyDo? ¿Tiene que implementar un nuevo punto final? ¿Necesitas configurar un servicio? Comencemos con the [Developer Guide](docs/developers/developer_guide.md)



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

