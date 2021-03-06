# BBB Background

[BBB](https://bigbluebutton.org/) is an OpenSource video conferencing tool. Per se, an installation uses a single server node. It can be extended though by the [scalelite load balancer](https://github.com/blindsidenetworks/scalelite/) and the [Greenlight user Management](https://github.com/bigbluebutton/greenlight/).

For an excellent user guide please refer to: https://soethe.net/bigbluebutton/

## This set-up (description)

This set-up makes use of a set of BBB instances enhanced by a scalelite and a greenlight installation. The entire application has been configured to run on [bwCloud](https://www.bw-cloud.org/), an OpenStack-based cloud installation operated by the universities from Baden-Württemberg.

!> **WARN** This documentation is preliminary. It is primarily focused on our production deployment in bwCloud but continuously upgraded towards a more generic documentation.

Roughly sketched overview:

- BBB installation in the cloud
- 10 BBB Instances (8 cores, 16 GB) with Scalelite, Moodle Plug-in, Prometheus Monitoring, Greenlight+LDAP and 40Gbit network
- Virtual machine deployment via Ansible
  - Operating system CoreOS Container Linux
  - A container orchestrator on every machine
- Deployment of the actual application via orchestrator (Rancher 1.6) using docker-compose
