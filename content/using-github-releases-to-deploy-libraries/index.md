---
title: "Using github releases to deploy libraries"
description: "I recently joined Sense Labs and having setup a static analysis build server a little while ago with Jenkins and Infer I decided to do the…"
date: "2016-08-07T19:04:33.861Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/using-github-releases-to-deploy-libraries-28ad83ce2689
redirect_from:
  - /using-github-releases-to-deploy-libraries-28ad83ce2689
---

I recently joined [Sense Labs](http://www.sense-labs.com/) and having setup a [static analysis build server](https://medium.com/@Rapchik/static-analysis-build-server-for-android-with-jenkins-using-docker-bda888d4b34e#.z9qj7sbai) a little while ago with Jenkins and Infer I decided to do the same for Sense.

At Sense we have multiple product teams for Android and iOS working on different products and a core platform team which works on a shared framework used across the company with the core written in C++ and a wrapper on top in Java and Objective-C.

The teams already had an infrastructure in place to automatically build the platform libraries but it still required a manual copy/paste phase to copy the frameworks into the Android/iOS projects. As I hate any and everything manual I decided to automate the process to make it a bit more fluid.

The approach was to use git tags to specify the version number of a new release and use that as a trigger to automatically generate a github release. Jenkins would build Debug and Release version of the frameworks, bundle them in a zip file and use a changes.md file to create a github release. The product teams can download the library by specifying the required version number (tag) in Gradle/info.plist for their project.

To be able to upload and create a release we needed to have a bot account with write permission to the repo. To make it more secure we inject the oAuth token generated from github as an environment variable in the build process and specify it to be a password so that we can enable masking to stop it from appearing in plain text in logs.

To differentiate between release and pre-release builds we use a naming convention and if the tag contains “rc” we understand it might not be stable and hence mark it as a pre-release when creating the release. To make it even nicer, you can generate the doxygen documentation for releases and commit it to gh-pages to automatically upload documentation.

> [python jenkins\_make\_release.py $ZIP\_FILE\_TO\_UPLOAD](https://github.com/umarniz/GithubReleaseSystem/blob/master/jenkins_make_release.py)

Now that we are automatically building the core library, how do we automate the process for the product teams for Android and iOS to download a version they need? To allow this for Android I created a groovy script in the gradle file and attached it to the compile task so that is platform agnostic. You can see the [android build.gradle here](https://github.com/umarniz/GithubReleaseSystem/blob/master/android_build.gradle).

For iOS, I created a python script as every Mac would have a python 2.x installation and added a build phase to download the correct version specified in the info.plist file and download the Debug/Release version according to the current configuration. The run script should be placed on the top with a similar code:

> plist=${PROJECT\_DIR}/${INFOPLIST\_FILE}

> sensesdkversion=$(/usr/libexec/PlistBuddy -c “Print SENSE\_SDK\_VERSION” “${plist}”)

> if \[\[ “${sensesdkversion}” == “” \]\]; then

> echo “No Sense SDK Version defined in $plist. Please set a version before building”

> exit 2

> fi

> python ./ios\_download\_sense\_sdk.py “${sensesdkversion}” “${CONFIGURATION}”

This looks for SENSE\_SDK\_VERSION in info.plist and tries to find that tag in Github Releases and download the respective version + configuration (Debug/Release) required for the project before we start building the project.

Having a Debug build allows us to profile and step into the source code of the framework without having the source files. This makes debugging and profiling the library a magnitude easier and allows the product teams to pin point the errors and fix the errors they need to fix for the shared framework themselves.

To be able to read a release from Github for private projects you would need a read only access token, for which we created another bot that has read only permissions for the library repos and generate an access token that we can keep in the repo. We use this access token in the build.gradle/iOS Python script for downloading the release.

Overall I really like how flexible github releases are and how powerful they can be for having an in-house release system. You can [checkout the code on github](https://github.com/umarniz/GithubReleaseSystem).
