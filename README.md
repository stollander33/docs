
# RAPyDo framework

A containers-based framework to develop your HTTP-API with Flask on Python3 with optional web interface written with Angular. RAPyDO also include services like relational databases (PostgreSQL, MariaDB), graph database (Neo4j), no-sql database (MongoDB) and Celery for asynchronous operations (based on RabbitMQ, also included in the box).

The name is an acronymous for:

- `R`ESTful `A`PI
- on a `Py`thon server
- based on a `Do`cker ecosystem



## Quick start

RAPyDo can be approached from two very different perspectives: the user and the developer.

The user usually interact with an already-existing RAPyDo-based project to deploy, monitor and manage one or more project installations. The developer creates a  RAPyDo-based project , customize the stack, implementens the endpoints.

Do you already have a RAPyDo-based project? You have to deploy it on a new server? Do you want to test an installation? Let's start with the [User Quick Start](docs/users/quick_start_users.md) and the [User Complete Guide](docs/users/user_guide.md)

Do you want to create a new RAPyDo-based project? You have to implement a new endpoint? You need to configure a service? Let's start with the [Developer Complete Guide](docs/developers/developer_guide.md)



## Why to use RAPyDo?

We don't know if you should. Along our journey into efficient HTTP API middleware towards distributed services we had to reach some level of stability within our environment.

Having the same tasks to be completed over and over we decided to develop RAPyDo to have a common base to all our RESTful HTTP API oriented projects. 
What we have so far is what helped us in speeding up setup and development process while keep track of all the solutions we found in all the problems we encountered in at least 5 years of italian/european projects experience.

So let's see together what we achieved so far!
(Please note that most of the features can be enabled/disabled as needed)

### modes

In every project there a set configurations that you would like to use to switch one from another quickly. Even if not the case there should be at least two modes: 

1. `debug` for working locally (verbose logs, dockerized services, auto reload and no reverse proxy)
2. `production` for deploying the same code in production (less logs, external production services rather than containers in the same node - we will have `swarm stacks` for this - `uwsgi` and a reverse proxy, all set to use `SSL` for free with Letsencrypt).

Modes are implemented simply a `YAML` file following the usual `compose` conventions.
You can create as many custom modes as you need.

The final configuration you use in every rapydo command is based on:

- A rapydo base configuration, gathering the best practices on the most commonly used services
- A common custom configuration across different modes in your project
- A custom file dedicated to a single mode

### modular

RAPyDo is of course containers oriented. This means that services can be easily added to be tested locally. We tested the most commons in our projects thus they are already integrated and all set:

- Postgres
- Redis
- MongoDB
- Elasticsearch
- Neo4J GraphDB
- RethinkDB
- RabbitMQ
- Celery tasks
- FTP
- Angular

More services can be added as long as you can provide a container image (official or not) for it; so basically always `:P`.

In production mode you may choose with a simple parameter `${SERVICE}_EXTERNAL` to switch to external existing services, without changing your endpoint code.

### swagger

The [OpenAPI standard]() has helped us many times to show to clients the experience of HTTP API services without even using a frontend. This is why every endpoint exists only if you create a dedicated folder in the `Swagger` definition. The content is parsed to activate the corresponding endpoints in your code and at the same time is added to the swagger specifications endpoint. 

Private endpoints can be skipped inside the public definitions. An entire endpoint can be easily turned off with a simple `SKIP` file put in the corresponding folder.
You can also associate a dependency of some endpoints to the existence of a variable of your configuration into your mode.

### rest classes

RAPyDO is Object oriented (thanks to the `Flask-Restful` plugin): each endpoint is mapped to a class and automatically configured. The class associated to an endpoint is provided in the swagger configuration (and then removed from the public view). One method can be mapped to multiple endpoints paths (e.g. if you need some aliasing).

### base endpoints

Helper endpoints are provided out of the box:

- `/api/status`
- `/api/specs`
- If you enable authentication:
- `/auth/login`
- `/auth/logout`
- `/auth/tokens`
- `/auth/profile`

They can be overriden or skipped.

### flask injections

The real first pain point we had to solve in our experience while working with Flask when containers were yet limited in their experience (at least a few years ago) was a dynamic injections of only the services configurated in the RAPyDO YAML files.

We created a set of defaults for each service. Each service can be activated with `${SERVICE}_ENABLE` or by adding the dependency on an already active service, and by leaving the defaults a container will be created for it and linked to the backend server/container and accessed by a simple `self.get_instance(my_service, **custom_args)` function call inside your endpoint code.

It doesn't matter what is your mode (internal with no credentials, or external with passwords), the code will remain the same!

NOTE: connections are kept globally in a pool to optimize the consumption of resources; you can force an instance to be recreated.

### orm

Each service may be used as an ORM, if the equivalent python library exists. The base models and the custom models are pre loaded into the service objects at server startup.

This means that you can do into your code:

```python
mongo = self.get_instance('mongo')
mongo.MyModel(field1='yes', field2=False).save()
```

Where `MyModel` was defined inside your project custom `models/mongo.py` file.

For each service base models are provided to describe a User/Role approach to authentication.

### logging

Logging has been made super easy. We added VERY_VERBOSE and VERBOSE to the standard sets of debug levels. You can activate colors, pretty print objects, and ultimately redirect containers logs into elasticsearch through LogSprut

### asynchrounous tasks

Celery tasks can be easily activated to be able to launch and control from any endpoint async tasks.
Tasks can be monitored directly from endpoints or from the `flower` UI.
Tasks can save progress, return status, send emails.

### unittests

`py.tests` is supported by default.
A unittest is a class in a separated `tests` folder, where you extend the existing base class from where you inherit methods to authenticate and handle tokens.

### containers builds

There is a set of already battle tested docker images.
All of them have been tuned in time, e.g. the nginx proxy base configuration
already considers all the best practices (dhelman, tls, certificates, etc.).
You can customize them or add new ones as well.

### security

Any service available in RAPyDo as an ORM can be used as authentication for the system, you just need to switch the dedicated variable `AUTH_SERVICE`.

Oauth2 and 2 factor authentication is already integrated (through TOTP).
The nginx reverse proxy has been tested in many production system.

### interfaces

A set of interfaces can be launched as containers to help with many services:

- flower (for celery)
- swaggerui
- adminer for SQL servers
- mongo express
  are among the ones we already use daily.

### highly customizable

Starting from the base we provide you can always add your custom operations and configurations.

We also allow a `.projectrc` to be used to set as default some custom host-dependent configuration that should not be saved in the repo.

### frontend framework

`Angular` is already integrated as base framework for the frontend part.
The base authentication (profile, change password, reset password, session lists, JWT tokens) endpoints are already tested inside the base TS code.

In debug mode the framework is served on nodejs/webpack while in production the static dist is built at startup time by the related container entrypoint.

NOTE: we are looking for `react` to be integrated as well!

### quick

When you ask for new endpoints with `rapydo template`, you receive things for free:

1. swagger definition example for a `GET` method
2. the base class implementing the endpoint already linked to swagger
3. a base service (postgres/sqlalchemy) injection example
4. unittest template in the right folder

### multi projects

The same repository can host different projects. This action came handy quite a lot of times when a similar project has to be maintaned in the same repo by the same people.

If the project is only one (the usual situation) it is used as default.

To switch projects you can use the `--project` command line parameter or set it inside the `projectrc` file.

