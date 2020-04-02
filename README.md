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

