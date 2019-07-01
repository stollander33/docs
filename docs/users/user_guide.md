### *Work in progress...*



# Upgrade to a new version

We assume that you already have a working deployment of your project (let's name it YOUR_PROJECT) based on version X and you want to upgrade to the new version Y.

1. #### stop the current stack

   `rapydo remove`

2. #### switch your git branch to the new version X

   `git fetch && git checkout X`

3. #### upgrade / update your RAPyDo controller

Probably the new version also requires a new version of RAPyDo. You can verify it by using the following command:

`rapydo version`

This command will give you in output:

​	a. the installed RAPyDo version

​	b. the version of YOUR_PROJECT

​	c. the required RAPyDo version

In this way you will be able to know if a new RAPyDo version is required or not and if you already meet the requisite.



Output examples:

> rapydo: 0.6.7	YOUR_PROJECT: Y	required rapydo: 0.6.7

You already have the required RAPyDo version. Probably you want to upgrade the controller anyway:

`rapydo install --git RAPYDO_VERSION`

You can also use the *auto* flag to let RAPyDo understand by itself which version is required :

`rapydo install --git auto`

Continue this guide by following step [4. update your submodules](user_guide.md#4a-update-your-submodules)



If the version requirement is unmet the output will be something like this:

> rapydo: 0.6.6	sci: 0.6.6	required rapydo: 0.6.7
>
> This project is not compatible with the current rapydo version (0.6.6)
>
> Please upgrade rapydo to version 0.6.7 or modify this project
>
> rapydo install --git 0.6.7

You can follow the suggestion or install the new controller by using the *auto* flag `rapydo install --git auto`

Continue this guide by following step [4b. upgrade your submodules](user_guide.md#4b-upgrade-your-submodules)



#### 	4a. update your submodules

This step is only required if you DIDN'T upgrade your RAPyDo version because the new version or YOUR_PROJECT do not require a new version of RAPyDo. In this case you will have to update your submodules and your project branch:

`rapydo update`

To update your submodules only: `rapydo update -i main`

Consider the opportunity to rebuild your containers build with: `rapydo build`



#### 	4b. upgrade your submodules

This step is only required if you upgraded RAPyDo to a new version, in this case you have to switch the version of all the submodules. To switch all your submodules to the new version:

`rapydo init`

This step will require some time since the new version will require new builds. You can skip the building phase here an postpone it on the next step by using the *no-build* flag (`rapydo init --no-build`)

5. #### Upgrade completed

Your upgrade procedure is completed, your are now able to restart your stack

`rapydo start`

Please not that this step could require new builds and take some time (first time only)

#### Post-upgrade issues:

- RAPyDo 0.6.7 upgraded the PostgreSQL version from 10.7 to 11.4. Databases created with psq10 are not compatible with psq11 and your container will fail to start with the following error:

  > FATAL:  database files are incompatible with server
  >
  > DETAIL:  The data directory was initialized by PostgreSQL version 10, which is not compatible with this version 11.4.

  You have two options to fix the error:

  - If your database is only used to store sessions you can simply destroy the database volume (an empty database will be automatically re-created at startup)

    `rapydo remove`

    `docker volume rm YOUR_PROJECT_sqldata`

    `rapydo start`

    In case of issues with the automatic init procedure your backend will be able to connect to PostgreSQL but the following error will be raise when trying to read data:
  
    >  Backend database is unavailable
  
    If this error occurs you can fix by:
  
    1. enter your backend container (`rapydo shell backend`)
    2. destroy and re-create the database (`restapi clean && restapi init`)
    3. exit the backend container (`exit`)
    4. restart the services (`rapydo restart`)
  
  - If you cannot lose your data, please refer to the [Official Upgrading Guide](https://www.postgresql.org/docs/11/upgrading.html)

