---
title: "Static analysis build server for Android with Jenkins using Docker"
description: "We have recently been working with quite a few simultaneous android projects at Nerdiacs so I decided to setup Jenkins on one of our…"
date: "2016-01-17T08:22:13.280Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/static-analysis-build-server-for-android-with-jenkins-using-docker-bda888d4b34e
redirect_from:
  - /static-analysis-build-server-for-android-with-jenkins-using-docker-bda888d4b34e
---

We have recently been working with quite a few simultaneous android projects at [Nerdiacs](http://www.nerdiacs.com/) so I decided to setup [Jenkins](https://jenkins-ci.org/) on one of our servers. The end goal of this build server is to be able to handle multiple Android SDKs, produce release builds with Lint and perform static code analysis using [Infer](http://fbinfer.com/) by Facebook.

Now firstly, we will setup Jenkins inside a [Docker](https://www.docker.com/) container as this allows us to just copy/paste the entire environment running Jenkins to any other server should we need to scale it in the future. The only pre-requisite for setting this up is to make sure you have Docker installed on the server you want to run, as everything else would run inside the docker container.

Now, first get the Docker image by cloning [this repo](https://github.com/umarniz/Jenkins-Android-Infer).

> git clone [https://github.com/umarniz/Jenkins-Android-Infer.git](https://github.com/umarniz/Jenkins-Android-Infer.git)

Second, we build the docker image by going into the repo directory and running the command:

> docker build -t jenkins\_android .

The final character ‘.’ here represents that we want to build the repo from the current directory. Docker looks for a “Dockerfile” in the current directory and uses the instructions in it to build the image. If you want to rename the image you can add “-t <image\_name>” before the dot. This will take a bit of time as Docker will download all dependencies and build them in the container.

Once the image is built successfully you can see the images you have installed on the system by typing.

> docker images

This should give you a list of images that are installed on the system and you should see jenkins\_android in the list.

Now that you have the image ready, lets run it:

> docker run -p 9090:8080 -v /var/jenkins:/var/jenkins\_home jenkins\_android

Now thats a handful of commands, lets break it down to see what its doing.

-   **docker run —** If i need to explain this, this tutorial might not be for you
-   **\-p 9090:8080 —** The -p command is used to forward ports, first one being the port on your server and the second one for the docker container. As Jenkins runs on port 8080 we are forwarding to port 9090, so you should be able to access Jenkins at yourserver.com:9090
-   \-**v /var/jenkins:/var/jenkins\_home —** The -v command is for binding a volume where the path **/var/jenkins** will be the directory on your server and **/var/jenkins\_home** will be inside the docker container. Simply put all files you (or jenkins) create in the docker container at /var/jenkins\_home will also be accessible at /var/jenkins on your server!

Congratulations you have successfully ran a Jenkins image inside a Docker container on your server! Check it out at yourserver.com:9090

---

You have Jenkins setup to be able to build android projects but it still needs configuration to be able to build Android projects, so lets get started.

Firstly, we need to add the brilliant [SDK Manager](https://github.com/JakeWharton/sdk-manager-plugin) gradle plugin to your project. The SDK Manager plugin downloads any SDK/Build Tools and even emulators that might be needed to build your project! This frees us of having to setup an Android SDK and automatically checks for anything that is needed every time you do a gradlew clean.

Make sure that you are not committing local.properties in your repo and that it is in your .gitignore, else the plugin will fail to run as gradle will try to find the Android SDK in the directory specified in local.properties which would be your computer.

Secondly, we need to add a keystore so that gradle can automatically release signed builds. I will not go in the depths of how to create a keystore as this is much better covered [here](http://developer.android.com/tools/publishing/app-signing.html).

Now that our project is ready to be built, lets setup Jenkins to get building. On the Jenkins main page clink on ‘Manage Jenkins’ on your left and goto the ‘Manage Plugins’ page. On the Manage Plugins page, first select the ‘Available’ tab item and then search and install the ‘Gradle plugin’ and ‘Bitbucket integration’ if your project is on Bitbucket.

Search for any other integrations you want, there are lots of fun plugins like ‘Google Play Android Publisher’, ‘Slack Integration’, etc.

Once you have the Gradle plugin installed, goto the main Jenkins page and create a new item and make a Freestyle project. You can configure your project first with whatever SCM you prefer, I used Bitbucket for our projects.

In the Build Triggers I have the server setup to poll the SCM every 5 minutes for a change. There are better methods but as this works, I never bothered changing it. To do this under Build Triggers, select Poll SCM and in the schedule paste this:

> H/05 \* \* \* \*

The ? icon on the right gives a detail on the syntax if you want to configure your polling time differently.

Now lets do the actual building. Under Build, ‘Add a Build Step’ for ‘Execute Shell’. We want to run these 3 commands:

> ./gradlew clean  
> infer — ./gradlew build  
> ./gradlew assembleRelease

Keep in mind that the infer step is very time consuming, specially on the first run and might be a better idea to do after you have done a couple of builds without infer.

Now that we are able to do release builds on the server, we would need to get the APKs from the output folder too right?

Todo this, add a post-build step for ‘Archive the artifacts’. In the Files to archive you want:

> \*\*/\*.apk

Assuming that the build.gradle is in the root folder, as the \*\* before / specifies that this we are looking for APKs 1 directory inside the root. It would change if your folder structure is different. This would get us all the \*.apks in the folder, but we don’t want all the \*.apks.

There is still the un-aligned APK that is built with the final zip-aligned release build. To ignore that we just add:

> \*\*/\*unaligned\*

to the ‘Excludes’ input (If you can’t see Excludes, click on Advanced).

And we are done!

You have a build system that runs Infer and Lint on your codebase and once its successful and free of all errors (even Lint ones) you get a bad-ass APK out of the system that’s wearing kevlar and is ready to cap any bug that comes near it!
