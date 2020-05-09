---
title: "fail2ban with Rancher Load Balancer (HAProxy)"
description: "This is a quick guide on how to make fail2ban work with Rancher Load Balancer running in a Cattle environment. The reason this specific…"
date: "2020-05-09T11:16:14.944Z"
categories: []
published: false
---

This is a quick guide on how to make fail2ban work with Rancher Load Balancer running in a Cattle environment. The reason this specific setup is a bit different from the usual fail2ban configuration is 2 folds:

1.  HAProxy is running in a docker container in a Cattle managed network which means that the incoming packets are [Forward packets and not Input packets](https://stackoverflow.com/a/23436938) for iptables.
2.  The way Cattle handles forwarding is a [bit ugly](https://github.com/rancher/rancher/issues/10387) and hence does not allow custom fail2ban rules
3.  HAProxy is running a docker image with server time in UTC and your servers might be in a different time zone.

---

In your /etc/fail2ban/jail.local change the \[DEFAULT\] configuration:

```
[DEFAULT]
...
chain = INPUT
...
```

TO

```
[DEFAULT]
...
chain = CATTLE_FORWARD
...
```

Change your time zone to
