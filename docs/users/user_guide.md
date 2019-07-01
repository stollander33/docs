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
> Please upgrade rapydo to version 0.6.7 or modify this project
>
> rapydo install --git auto

Upgrade your controller by executing the suggested command.

Continue this guide by following step [4b. upgrade your submodules](user_guide.md#4b-upgrade-your-submodules)



#### 	4a. update your submodules

This step is only required if you DIDN'T upgrade your RAPyDo version because the new version or YOUR_PROJECT do not require a new version of RAPyDo. In this case you will have to update your submodules and your project branch:

`rapydo update`

To update your submodules only: `rapydo update -i main`

Consider the opportunity to rebuild your containers build with: `rapydo build`



#### 	4b. upgrade your submodules

This step is only required if you upgraded RAPyDo to a new version, in this case you have to switch the version of all the submodules. To switch all your submodules to the new version:

`rapydo init`

This step will require some time since the new version will require new builds. You can skip the building phase here an postpone it on the next step by using the `--no-rebuild` flag.	

5. #### Upgrade completed

Your upgrade procedure is completed, your are now able to restart your stack

`rapydo start`

Please not that this step could require new builds and take some time (only the first time)