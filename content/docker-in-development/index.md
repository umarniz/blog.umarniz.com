---
title: "Docker in development"
description: "Having used Docker on and off for lots of small dev environments in the past couple of years, containers have become by goto solution for…"
date: "2017-01-07T15:29:47.885Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/docker-in-development-339110a03c22
redirect_from:
  - /docker-in-development-339110a03c22
---

Having used Docker on and off for lots of small dev environments in the past couple of years, containers have become by goto solution for developing any new micro service.

After joining [Sense Health](http://www.sense-health.com/) a few months ago, I have started building a large scale backend technology that involves making several new micro services but more importantly it requires heavily modifying several existing services. In this post I will try to discuss lessons learnt building containers for 6 services and 5 horizontally scaled databases all orchestrated with docker-compose files.

### #1 docker-compose can’t wait for container ‘ready’

![Track has not been initialised](./asset-1.png)

docker-compose has no built-in mechanism to [wait for a service](https://docs.docker.com/compose/startup-order/) to fully initialise before starting another service. You can define the order each container is started in by using ‘[depends\_on](https://docs.docker.com/compose/compose-file/#dependson)’ in the compose image but it does not guarantee for a service to be ready before the next container is started.

This causes problems specially with databases as we need the databases to be initialised and fully loaded before we can start a service. To help resolve this I built a helper program that would be initialised by a bash script and would start the service only after all required services have been initialised.

As I am dockerizing a legacy service and don’t want to do massive modifications to the code I built an external program but in the new micro-services that I am developing, have built in state management to wait for all required dependencies to be online and ready before initialising itself.

### #2 unit test containers

![Which container has the ball?](./asset-2.jpeg)

I was dockerizing services that already were running in production and were only actively developed by very few people in my company. I noticed even though a lot of people had the skills to work on these services they did not go hands on as there was a high likelihood of breaking things and the setup steps for building and testing the services were quite complex and intimidating for new developers to follow.

To make sure that dockerizing the entire infrastructure would be actually useful to everyone, my strategy was to build containers for each service that would run as unit tests in default mode and report success/failure of all tests on exit. With environment variables the same container would run in production mode with a different configuration.

This approach gave people an actual reason to work with docker. People trying to fix a bug in the service noticed they could actually test their fix with a single line ‘docker-compose up’ and as it was solving a problem I noticed adoption go up for docker in the company and people reading up more and more on it.

### #3 docker-entrypoint

![Be lazy. Don’t do if its already done.](./asset-3.jpeg)

Another important thing during development is the need for configuring or initialising databases with backups. Initialising a docker container with a custom configuration is quite straightforward if you build your own docker image but with lots of continuously evolving databases required for our projects I did not want to create a modified docker image for each database.

While developing the services I noticed that most official docker images of databases have a special [docker-entrypoint-initdb.d](https://github.com/docker-library/mariadb/blob/c31e557088187e8ac7a7d4b13e7bd0c052386f94/10.1/docker-entrypoint.sh#L151) folder from which it can process restores, run sql/cql files to initialise tables/keystores. This allowed me to use official docker images for mariadb, postgres and cassandra and mount the entrypoint folder with the required configuration or scripts for setting up the database.

I used git-lfs to store encrypted binary backups of our testing servers and mounted the volume on run time. Alphabetically naming them also avoided missing table errors as the initialisation would always create they keyspace and tables and only after that would we do the actual restore of the database.

---

It took me less than 2 weeks to tame years of undocumented dependencies into lots of clean containers with multiple docker-compose files for building the services in different modes. We are still using our existing pipeline for deploying services to production servers for now but with docker driven development the speed of development of all our services have sped up greatly with contributions c0ming from the entire team.
