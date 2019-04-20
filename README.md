<!--

    Sonatype Nexus (TM) Open Source Version
    Copyright (c) 2018-present Sonatype, Inc.
    All rights reserved. Includes the third-party code listed at http://links.sonatype.com/products/nexus/oss/attributions.

    This program and the accompanying materials are made available under the terms of the Eclipse Public License Version 1.0,
    which accompanies this distribution and is available at http://www.eclipse.org/legal/epl-v10.html.

    Sonatype Nexus (TM) Professional Version is available from Sonatype, Inc. "Sonatype" and "Sonatype Nexus" are trademarks
    of Sonatype, Inc. Apache Maven is a trademark of the Apache Software Foundation. M2eclipse is a trademark of the
    Eclipse Foundation. All other trademarks are the property of their respective owners.

-->
# Nexus Repository Composer Format

[![Build Status](https://travis-ci.org/sonatype-nexus-community/nexus-repository-composer.svg?branch=master)](https://travis-ci.org/sonatype-nexus-community/nexus-repository-composer) [![Join the chat at https://gitter.im/sonatype/nexus-developers](https://badges.gitter.im/sonatype/nexus-developers.svg)](https://gitter.im/sonatype/nexus-developers?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![DepShield Badge](https://depshield.sonatype.org/badges/sonatype-nexus-community/nexus-repository-composer/depshield.svg)](https://depshield.github.io)

# Table Of Contents
* [Developing](#developing)
   * [Requirements](#requirements)
   * [Building](#building)
   * [Travis CI](#travis-ci)
* [Using Composer with Nexus Repository Manger 3](#using-composer-with-nexus-repository-manager-3)
* [Installing the plugin](#installing-the-plugin)
   * [Temporary Install](#temporary-install)
   * [(more) Permanent Install](#more-permanent-install)
   * [(most) Permament Install](#most-permanent-install)
* [Running](#running)
* [Notes](#notes)
    * [Persistent Data](#persistent-data)
* [The Fine Print](#the-fine-print)
* [Getting Help](#getting-help)
* [Composer Plugin](#composer-plugin)

## Developing

### Requirements

* [Apache Maven 3.3.3+](https://maven.apache.org/install.html)
* [Java 8+](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* Network access to https://repository.sonatype.org/content/groups/sonatype-public-grid

Also, there is a good amount of information available at [Bundle Development](https://help.sonatype.com/display/NXRM3/Bundle+Development)

### Building

To build the project and generate the bundle use Maven

    mvn clean package

If everything checks out, the bundle for Composer should be available in the `target` folder

### Travis CI

This project is built automatically with Travis CI, and you should be able to see build results on your commits, as well
as directly on [Travis CI](https://travis-ci.org/sonatype-nexus-community/nexus-repository-composer/).

#### Build with Docker

To build a docker image from the Docker file you can use this command:

`docker build -t nexus-repository-composer:0.0.2 .`

The following optional variables can be used when building the image:

- NEXUS_VERSION: Version of the Nexus Repository Manager
- NEXUS_DOWNLOAD_URL: Download URL for Nexus Repository, alternative to using `NEXUS_VERSION` to download from Sonatype
- NEXUS_DOWNLOAD_SHA256_HASH: Sha256 checksum for the downloaded Nexus Repository Manager archive. Required if `NEXUS_VERSION`
 or `NEXUS_DOWNLOAD_URL` is provided

#### Run as a Docker container

`docker run -d -p 8081:8081 --name nexus nexus-repository-composer:0.0.2`

For further information like how to persist volumes check out [the GitHub repo for our official image](https://github.com/sonatype/docker-nexus3).

The application will now be available from your browser at http://localhost:8081

## Using Composer With Nexus Repository Manager 3

[We have detailed instructions on how to get started here!](docs/COMPOSER_USER_DOCUMENTATION.md)

## Installing the plugin

There are a range of options for installing the Composer plugin. You'll need to build it first, and
then install the plugin with the options shown below:

### Temporary Install

Installations done via the Karaf console will be wiped out with every restart of Nexus Repository. This is a
good installation path if you are just testing or doing development on the plugin.

* Enable Nexus console: edit `<nexus_dir>/bin/nexus.vmoptions` and change `karaf.startLocalConsole`  to `true`.

  More details here: [Bundle Development](https://help.sonatype.com/display/NXRM3/Bundle+Development+Overview)

* Run Nexus' console:
  ```
  # sudo su - nexus
  $ cd <nexus_dir>/bin
  $ ./nexus run
  > bundle:install file:///tmp/nexus-repository-composer-0.0.2.jar
  > bundle:list
  ```
  (look for org.sonatype.nexus.plugins:nexus-repository-composer ID, should be the last one)
  ```
  > bundle:start <org.sonatype.nexus.plugins:nexus-repository-composer ID>
  ```

### (more) Permanent Install

For more permanent installs of the nexus-repository-composer plugin, follow these instructions:

* Copy the bundle (nexus-repository-composer-0.0.2.jar) into <nexus_dir>/deploy

This will cause the plugin to be loaded with each restart of Nexus Repository. As well, this folder is monitored
by Nexus Repository and the plugin should load within 60 seconds of being copied there if Nexus Repository
is running. You will still need to start the bundle using the karaf commands mentioned in the temporary install.

### (most) Permanent Install

If you are trying to use the Composer plugin permanently, it likely makes more sense to do the following:

* Copy the bundle into `<nexus_dir>/system/org/sonatype/nexus/plugins/nexus-repository-composer/0.0.2/nexus-repository-composer-0.0.2.jar`
* Make the following additions marked with + to `<nexus_dir>/system/org/sonatype/nexus/assemblies/nexus-core-feature/3.x.y/nexus-core-feature-3.x.y-features.xml`

   ```
         <feature prerequisite="false" dependency="false">nexus-repository-rubygems</feature>
   +     <feature prerequisite="false" dependency="false">nexus-repository-composer</feature>
         <feature prerequisite="false" dependency="false">nexus-repository-gitlfs</feature>
     </feature>
   ```
   And
   ```
   + <feature name="nexus-repository-composer" description="org.sonatype.nexus.plugins:nexus-repository-composer" version="0.0.2">
   +     <details>org.sonatype.nexus.plugins:nexus-repository-composer</details>
   +     <bundle>mvn:org.sonatype.nexus.plugins/nexus-repository-composer/0.0.2</bundle>
   + </feature>
    </features>
   ```
This will cause the plugin to be loaded and started with each startup of Nexus Repository.

## Running

To run, binding the exposed port 8081 to the host.

```
$ docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

To test:

```
$ curl -u admin:admin123 http://localhost:8081/service/metrics/ping
```

## Notes

* Default credentials are: `admin` / `admin123`

* It can take some time (2-3 minutes) for the service to launch in a
new container.  You can tail the log to determine once Nexus is ready:

```
$ docker logs -f nexus
```

* Installation of Nexus is to `/opt/sonatype/nexus`.

* A persistent directory, `/nexus-data`, is used for configuration,
logs, and storage. This directory needs to be writable by the Nexus
process, which runs as UID 200.

* There is an environment variable that is being used to pass JVM arguments to the startup script

  * `INSTALL4J_ADD_VM_PARAMS`, passed to the Install4J startup script. Defaults to `-Xms1200m -Xmx1200m -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=${NEXUS_DATA}/javaprefs`.

  This can be adjusted at runtime:

  ```
  $ docker run -d -p 8081:8081 --name nexus -e INSTALL4J_ADD_VM_PARAMS="-Xms2g -Xmx2g -XX:MaxDirectMemorySize=3g  -Djava.util.prefs.userRoot=/some-other-dir" sonatype/nexus3
  ```

  Of particular note, `-Djava.util.prefs.userRoot=/some-other-dir` can be set to a persistent path, which will maintain
  the installed Nexus Repository License if the container is restarted.

* Another environment variable can be used to control the Nexus Context Path

  * `NEXUS_CONTEXT`, defaults to /

  This can be supplied at runtime:

  ```
  $ docker run -d -p 8081:8081 --name nexus -e NEXUS_CONTEXT=nexus sonatype/nexus3
  ```

### Persistent Data

There are two general approaches to handling persistent storage requirements
with Docker. See [Managing Data in Containers](https://docs.docker.com/engine/tutorials/dockervolumes/)
for additional information.

  1. *Use a docker volume*.  Since docker volumes are persistent, a volume can be created specifically for
  this purpose.  This is the recommended approach.

  ```
  $ docker volume create --name nexus-data
  $ docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
  ```

  2. *Mount a host directory as the volume*.  This is not portable, as it
  relies on the directory existing with correct permissions on the host.
  However it can be useful in certain situations where this volume needs
  to be assigned to certain specific underlying storage.

  ```
  $ mkdir /some/dir/nexus-data && chown -R 200 /some/dir/nexus-data
  $ docker run -d -p 8081:8081 --name nexus -v /some/dir/nexus-data:/nexus-data sonatype/nexus3
  ```

## The Fine Print

It is worth noting that this is **NOT SUPPORTED** by Sonatype, and is a contribution of ours
to the open source community (read: you!)

Remember:

* Use this contribution at the risk tolerance that you have
* Do NOT file Sonatype support tickets related to Composer support in regard to this plugin
* DO file issues here on GitHub, so that the community can pitch in

Phew, that was easier than I thought. Last but not least of all:

Have fun creating and using this plugin and the Nexus platform, we are glad to have you here!

## Getting help

Looking to contribute to our code but need some help? There's a few ways to get information:

* Chat with us on [Gitter](https://gitter.im/sonatype/nexus-developers)
* Check out the [Nexus3](http://stackoverflow.com/questions/tagged/nexus3) tag on Stack Overflow
* Check out the [Nexus Repository User List](https://groups.google.com/a/glists.sonatype.com/forum/?hl=en#!forum/nexus-users)

## Composer Plugin
The composer plugin `elendev/nexus-composer-push` (https://github.com/Elendev/nexus-composer-push) provide a composer command to push to a Nexus Repository using this plugin.
