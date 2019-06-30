## Quick Start for Users

This a quick start guide, if you are interested in a more comprehensive guide please refer to the [User Complete Guide](user_guide.md)

1. Make sure you meet the pre-requisites on your machine:
    * Python3.4+ (and `pip`) 
    * Git
    * Docker daemon/engine
2. install the rapydo controller
3. clone your project

```bash
git clone https://github.com/rapydo/example.git
cd example
cp templates/projectrc .projectrc <<< ?
```

3. Use the controller to startup your project

```bash
# check the status of your scaffold and
# download others repo from the rapydo framework as submodules
rapydo init
# launch the containers
rapydo start
```

Note: the main backend containers with Flask will be built now. It might take a few minutes, please be patient.

5. Launch the server in debug mode

```bash
# check the containers running
rapydo status
# open a shell in the current backend container
rapydo shell backend
# inside the container you can run a flask server in debug mode
$ restapi launch

[...]
# lots of logs
# if you need to stop it use CTRL+C
```

The example provides a simple Flask server connected to a postgresql db into another container.

6. Test your server from outside the containers

The port `8080` should be accessible now.
You may help your self with the famous `httpie` [python command line tool](https://httpie.org/) to query the API endpoints from another shell, and check the logs in the other one. It was already installed on your host together with the rapydo framework at the step 3.

Currently there is only one custom endpoint called `myendpoint`, plus all the others helpers already bundled

```bash
# test a normal endpoint with no authentication
http GET localhost:8080/api/myendpoint

# request a token to the server with a default account
# the account is available only in debug mode
http POST localhost:8080/auth/login username=user@nomail.org password=test

# save the token from the previous response
export TOKEN="..."

# call an authenticated endpoint helper, e.g. check the profile
http GET localhost:8080/auth/profile Authorization:"Bearer $TOKEN"
# "status": "Valid user"
```

7. check the swagger definitions

Your server automatically provides a description compliant to the [OpenAPI standard]() at the URI:
http://localhost:8080/api/specs

And you may launch a swagger interface container to access your current API endpoints description and play with it:

```bash
rapydo interfaces swagger

[...]
You can access swaggerui web page here:
http://localhost?docExpansion=none
```

8. Add a new endpoint

You can ask the framework to prepare templates to develop a new endpoint:

```bash
rapydo template anotherendpoint

[...]
rendered projects/example/backend/swagger/anotherendpoint/specs.yaml
rendered projects/example/backend/swagger/anotherendpoint/get.yaml
rendered projects/example/backend/apis/anotherendpoint.py
rendered projects/example/backend/tests/test_anotherendpoint.py
Scaffold completed
```

As you can see there is already a swagger configuration produced for you.

You can now already restart the server to reload the configuration and try the new endpoint:
```bash
http GET localhost:8080/api/anotherendpoint

# "Hello World!"
```

## see a live webinar

To get a better understanding of how this framework is being used in production for real projects, you may watch [this video](http://bit.ly/b2stage_webinar) recorded last year.

## clean up

To remove everything you created so far, you can use again the controller:
```bash
rapydo clean -rm
```
