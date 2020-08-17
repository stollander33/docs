## Quick Start for Users

This a quick start guide, if you are interested in a more comprehensive guide please refer to the [User Complete Guide](user_guide.md)

1. Make sure you meet the pre-requisites on your machine:
    * Python 3.6+ (and `pip`) 
    * Git
    * Docker daemon/engine
    
2. install the RAPyDo controller

    Install the latest version: `sudo pip3 install --upgrade rapydo-controller`

    You have now the executable `rapydo` (you can try a `rapydo --version`). Your project could require a different version, in this case you will be able to install the right version once configured your project

    NOTE: python install binaries in `/usr/local/bin`. If you are not the admin/`root` user (i.e. you do not install with `sudo`) then a virtual environment is created and you may find the binary in `$HOME/.local/bin`. Make sure that the path is in your `$PATH` variable, otherwise you will end up with `command not found`.

3. clone and initialize your project

```bash
git clone https://git.../your_project/example.git
git checkout your_branch
rapydo init
```

3. Customize your `.projectrc` file. By editing this file you can override all options of your project for your specific deployment (e.g. to set new secret passwords for deployed services)

4. If your project requires it, install the specific RAPyDo controller version

   You can verify the required version with `rapydo version`

   You can install the version required by your project with: `rapydo install`

5. Use the controller to start your project

```bash
# pull base images
rapydo pull
# launch the containers
rapydo start
```

5. Launch the server in debug mode

```bash
# check the containers are running
rapydo status
# start the backend default command in interactive mode
rapydo shell backend --default-command
# Please note that this command is equivalent to:
# rapydo shell backend --command "restapi launch"

[...]
# lots of logs
# if you need to stop it use CTRL+C
```

6. Test your server

The port `8080` should be accessible now, you can query it by using a a http client (curl, wget, `httpie` [HTTPie] (https://httpie.org/) from another shell, and checking the logs in the other one.

```bash
# test a normal endpoint with no authentication
http GET localhost:8080/api/status

# request a token to the server with a default account
# the account is available only in debug mode
http POST localhost:8080/auth/login username=user@nomail.org password=XXX

# save the token from the previous response
export TOKEN="..."

# call an authenticated endpoint helper, e.g. check the profile
http GET localhost:8080/auth/profile Authorization:"Bearer $TOKEN"
# "status": "Valid user"
```

7. check the swagger definitions

Your server automatically provides a description compliant to the [OpenAPI standard]() at the URI:
http://localhost:8080/api/swagger

And you may launch a swagger interface container to access your current API endpoints description and play with it:

```bash
rapydo interfaces swagger

[...]
You can access swaggerui web page here:
http://localhost?docExpansion=none
```



## Clean up

You can stop your containers with the command:

```
rapydo stop
```

To remove the containers by keeping all data:

```
rapydo remove
```

To remove containers and networks:

```
rapydo remove --networks
```

To remove everything you created so far, you can use again the controller:

```bash
rapydo remove --all
```

