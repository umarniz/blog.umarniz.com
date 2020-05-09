---
title: "Revamping a legacy backend #1"
description: "Infrastructure as Code"
date: "2018-01-04T08:56:17.983Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/revamping-a-legacy-backend-1-1429a4ce77cb
redirect_from:
  - /revamping-a-legacy-backend-1-1429a4ce77cb
---

### Infrastructure as Code

![A scene from The Matrix](./asset-1.jpeg)

At [Sense Health](https://www.sense-health.com/) over the past year I have been leading the technology team to revamp our core tech stack. Over the year we have changed our tooling heavily and introduced new processes to improve development speed, reduce bugs in our mobile applications and provide the right control to each domain expert (POs, Developers, Coaches, Content Writers).

This has involved a lot of new development which makes it relatively straightforward for most cases as old technologies are deprecated and new ones are built; but the backend for the organisation has a huge amount of code on which all legacy and current applications depend on.

The code for the backend runs on lots of servers distributed over multiple providers and with firewall configurations that are almost impossible to remember. These servers were setup years ago by developers who have already left the company and have tools installed on them by multiple engineers over a span of several years.

With several different monitoring tools and different configurations for each server, it would be almost impossible for us to debug problems fast enough for production to not be affected. The on-boarding for new backend developers was more or less learning by being on-call with a senior engineer and fixing bugs as they came up.

With a frail infrastructure for which none of the new engineers could confidently solve problems, we decided to do an overhaul.

![](./asset-2.png)

As we were doing the overhaul we decided to make sure the entire infrastructure is reproducible from scratch if required and that everything should be written down as code. To ensure this is possible, the first step we took was to use [Terraform](https://www.terraform.io/) to instantiate our servers.

Terraform allows you to write down your server configuration as a YAML file. This involves not only specifying the configuration of the server but also the network its associated with and the firewall rules applied to it.

Terraform also supports a [long list of providers](https://www.terraform.io/docs/providers/index.html) that means that moving from a local hosting provider to lets say the Google Cloud or AWS would not involve a lot of work but would just require a translation to the providers resources.

As our hosting provider supports OpenStack, we used the OpenStack provider to describe our infrastructure in Terraform which means we can instantly move our entire infrastructure to any provider that supports OpenStack!

To ensure the move has no impact to our existing customers, we duplicated our entire server stack using Terraform and redesigned the network communication between servers and reduced the complexity of firewall rules to decrease chances of human error.

![](./asset-3.png)

Now that we have the power to instantiate a server using code we need to define what softwares we want to install on this server.

To do this we decided to use [Ansible](https://www.ansible.com/). Ansible provides [playbooks](http://docs.ansible.com/ansible/latest/playbooks.html) which provide a series of steps to perform on a server. Playbooks are a YAML file that provide step by step instructions to perform on a server.

Ansible provides powerful tools to keep things modular and reusable using [Roles](http://docs.ansible.com/ansible/latest/playbooks_reuse_roles.html). Roles provide configuration and tasks in a package that can be included by a playbook. This means things like [installing an NGINX webserver](https://galaxy.ansible.com/geerlingguy/nginx/) or [setting up Redis](https://galaxy.ansible.com/geerlingguy/redis/) are already done for you by just including a role.

To ensure consistency across our entire stack, we created a custom role that was at the heart of all our servers. This installs an array of software to setup security, monitoring and maintenance tools for us.

Every time we deploy a new server for any reason at all, it is configured using Ansible and hence instantly shows up in our monitoring & alerting systems and allows users in the right LDAP group access to the server.

We use Ansible to setup all services on our servers and ensure that no manual changes are done to the servers by logging in but instead all configuration should be done through Ansible.

To further reduce potential sources of human errors, we have a single Sense backup and restore role that is attached to every server that has a database on it. This role is capable of automatically backing up every database we support at Sense and handles scheduling and uploading backups automatically.

The restore part of the script can be manually triggered with a specified version of the database that we use for custom tests and to ensure we have a fast response time in case of catastrophic failure.

---

With this mix of custom roles + putting the code for our entire infrastructure in a git repo gives us complete control over our infrastructure including version control!

This revamp allows us to describe our entire infrastructure as code and has made our lives much easier in managing our services and made it much easier to onboard new engineers to this stack as it has become quite self explanatory.

This makes instantiating new servers and provisioning services on them to be a piece of cake. In the next article, I explain how we [continuously deploy our code](https://hackernoon.com/revamping-a-legacy-backend-2-56d6c98df1f3) to ensure any engineer in the company can deploy to staging without support from the backend team.
