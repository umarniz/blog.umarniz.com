---
title: "Revamping a legacy backend #3"
description: "Going to Production"
date: "2018-06-11T07:06:01.144Z"
categories: []
published: true
canonical_link: https://medium.com/@Rapchik/revamping-a-legacy-back-end-3-4cb0d39f7fdd
redirect_from:
  - /revamping-a-legacy-back-end-3-4cb0d39f7fdd
---

**Going to Production**

![Cookie Production Line (Source: Unknown)](./asset-1.jpeg)

With an [infrastructure as code](https://hackernoon.com/revamping-a-legacy-backend-1-1429a4ce77cb) for our server provisioning and a [containerized environment for our APIs](https://hackernoon.com/revamping-a-legacy-backend-2-56d6c98df1f3) we are ready to migrate our production stack to the new container based setup. In this article I will explain our approach to production deployment and how we are ensuring high up-time of our legacy stack with minimal support.

Having tested our staging environment to be functional and being used for our deployments, we decided to do the final switch to production while minimizing down time.

Before we do the actual migration, we need to make sure we are alerted as soon as services go down or face any hiccups.

**Monitoring and Alerting**

For monitoring system health we have already been using [Prometheus](https://prometheus.io/) for our existing stack. After deploying Rancher we added some metrics from Rancher to our Prometheus stack to ensure we could monitor service health from Prometheus.

Secondly we added alerts to our Prometheus stack by using [Alert Manager](https://prometheus.io/docs/alerting/alertmanager/). This allows us to create alerts based on different thresholds and operations on metrics collected by Prometheus. With each alert, we connect a severity rating ranging from Low (Disk Space > 80%, CPU usage high on staging servers, etc) to Critical (Cluster down, Rancher Service down, etc).

These alerts are fed into our incident management tool, [OpsGenie](https://www.opsgenie.com/) where we have an on-call schedule for the back-end team to ensure all incidents are resolved in a timely manner. This process has allowed us to avoid problems before they occur by reacting to potential upcoming issues before our users are affected and in an unfortunate scenario of a critical failure to be alerted the instance the service fails.

![**Time to deploy!**](./asset-2.gif)

Now that we have monitoring and alerting setup, we are ready to deploy to production! We initially mirrored all our services to a production rancher environment and ran stress tests for a few days to ensure everything was working as expected. After several days of testing and fine tuning our stack by re-deploying images multiple times a day we found and fixed several problems related to cleaning up dangling images, disk space issues due to oversized logs, etc and were now ready to do the final production switch over.

To ensure minimum downtime, we started by migrating a small amount of our traffic to one of the rancher servers from our DNS. After a day of successful production traffic handled by the Rancher cluster we decided to do the full switch over with load balancing on 3 Rancher agents while still keeping the legacy server running as a backup if things went wrong.

A month of testing with this setup gave us full confidence that our Rancher stack was fully functional and shut down the old server marking completion of this migration!

**Wohoo! Done with the legacy stuff, now we have a successful Docker based stack that should work flawless right?**

Well, almost. Even though now we have multiple fail overs and a well scaled setup, we are still working with legacy applications that were never designed with such a stack in mind.

**Configuration Hell**

In the first 2 months of production deployment, we had multiple incidents all of them primarily coming from incorrect configuration.

Its a well known problem that handling configurations in distributed environments can be quite painful as there are multiple copies of the service running on different servers and they all need to have the same configuration. To solve this, we coupled our application configurations with our server configurations in Ansible and ensured that all configurations have a single source of truth, i.e Ansible.

While this might not the best solution as it requires a strict following of process, i.e never edit configuration files manually on the server and there are better solutions out there e.g [etcd](https://github.com/coreos/etcd) coupled with [confd](https://github.com/kelseyhightower/confd) which can ensure configurations to be in a single place, it was important to identify the root of the problem before jumping into the solution.

The root of our problems were 2 folded:

1.  **Most of our applications loaded configurations from files only on boot-up**

To explain why this is a bad practice for distributed systems, let me run you through a scenario.

Imagine you update all servers with changes to the configuration file from Ansible and re-start all services correctly. The server is running perfectly fine and everything is working as expected with the updated configuration.

Now a week later, someone from the ops team re-runs the Ansible playbook but in doing so, they accidentally run the Ansible playbook from a branch that does not contain the updated configuration file. When they deploy the incorrect configuration file to the server, the services running on this server aren’t affected as they loaded the configuration on boot-up and have the correct configuration in memory.

Now we are in an ugly situation of incorrect configuration on the server, but the correct configuration in memory and hence we are seeing no problems at this moment.

A week later, someone tries to release a new version of the application. At this moment the container has to be recreated with the new image and the incorrect configuration on the server is picked up by the application and the service starts failing.

---

This is causes the problem to be extremely nasty to debug. The developers are unaware whether their change caused the problem or not and the Dev Ops engineer who ran the Ansible playbook is completely unaware that their re-run caused the actual problem in the first place.

To solve this problem, instead of doing heavy changes to the code base itself (which could result in more problems due to it being a legacy code base), I added a one liner supervisor program that would restart the service whenever the configuration file was changed.

> command=bash -c “inotifywait -m -e modify /etc/service/service.conf | while read events; do supervisorctl restart SupervisorService; done”

This means that as soon as someone runs the Ansible playbook with the incorrect configuration, the service is restarted effectively pulling the latest configuration and causing the service to fail if its in an incorrect configuration.

Note: This solution could be extended by adding etcd & confd to avoid the problem created by running Ansible from an incorrect branch but to avoid spending more time then strictly required, we ensured that we only run Ansible playbooks for Rancher Agents from the Master branch.

**2\. No checks for inter-service connections**

With our monitoring and alerting setup we are able to detect problems on servers and services as soon as they occur and with multiple nodes in each cluster our back-end is quite resilient to services going down.

Even though this gives us complete confidence on each node and service itself, we are still missing data on inter-service connections. This means that even though we know MySQL is up and we know our service is also up, it doesn’t mean everything is running as expected. It could be that somehow our connection to MySQL is broken (bad firewall rule, incorrect credentials, etc).

As we are working with legacy code that did not have distributed deployment in mind, our services have no concept of health checks.

To improve reliance and to increase confidence of our services, I decided to add a Health Check endpoint to all our services. Again to ensure minimal interaction with existing code (to avoid new problems from arising), I spent a week to make an endpoint in all our micro-services that returns 200 or 400 status code and returns a JSON object with the milli-seconds it took to get a response from each services it relies on.

An example of these tests would be to get a default user from MySQL, get values using RPC from another service, simple HTTP request to a third party service, etc. If all responses were successful, it would return a status 200 and otherwise return status 400.

This gives us an extremely easy way to debug if the system is working well and also to debug if any issues occur. An example of debugging issues, one day we suddenly faced create user requests timing out, the problem had never been faced before and would have taken a long time to debug as no-one knew why this was happening in the first place. With multiple micro-services working together and complex database relationships, this problem might have taken a very long time to debug but with the status end-point we instantly discovered that 1 micro-service was taking 30 seconds to respond to requests allowing us to identify and resolve the issue in record time.

---

This has been a huge learning experience for us converting a legacy back-end with a frail infrastructure to an extremely robust fail-safe cluster. Since the deployment of this setup, and applying the changes explained above, we have successfully crossed 3 months with 0 major incidents.

Did this help you migrate your legacy back-end to a fail safe environment? Drop a comment and let me know, I would love to hear your story :)
