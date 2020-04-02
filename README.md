## BBB Background

[BBB](https://bigbluebutton.org/) is an OpenSource video conferencing tool. Per se, an installation uses a single server node. It can be extended though by the [scalelite load balancer](https://github.com/blindsidenetworks/scalelite/) and the [Greenlight user Management](https://github.com/bigbluebutton/greenlight/).

## This set-up (description)

This set-up makes use of a set of BBB instances enhanced by a scalelite and a greenlight installation. The entire application has been configured to run on [bwCloud](https://www.bw-cloud.org/), an OpenStack-based cloud installation operated by the universities from Baden-Württemberg.

### Infrastructure deployment

The current set-up consists of more than a dozen Virtual Machines in bwCloud. This set-up has been created through a set of ansible scripts and leads to a configuration consisting of the following entities:
* N virtual machines (with N being configurable in ansible)
* A Rancher server (Rancher version 1.6) running on one of these Virtual Machines.
* Rancher-agents running on the remaining N-1 Virtual Machines

### Application deployment

The application is again triggered by ansible and then actually deployed over Rancher. Here, we have defined a series of application stacks that for the entire set-up. These include:
* one stack per bbb instance (e.g. bbb-01) consisting of three docker containers
   * bbb: the actual bbb instance
   * bbb-exporter: a prometheus exporter providing monitoring data
   * lb: a load balancer providing entry points for bbb and mainly used for SSL termination
* bbb-greenlight consisting of 
   * greenlight: the actual greenlight container
   * lb-greenlight: a load balancer providing an entry point for greenlight and mainly used for SSL termination
 * bbb-scalelite realising the scalelite fuctionality consisting of the following containers
   * scalelite-api: the scalelite-api component as [described here](https://github.com/blindsidenetworks/scalelite/blob/master/docker-README.md)
   * scalelite-poller: the scalelite poller component as  [described here](https://github.com/blindsidenetworks/scalelite/blob/master/docker-README.md)
   * scalelite-redis: a redis instance used by scalelite as  [described here](https://github.com/blindsidenetworks/scalelite/blob/master/docker-README.md)
   * scalelite-registrator: a custom-built component that polls consul and registers all BBB instances with the load balancer; it further enables these servers.
   * The postgres instance required by Scalelite is contained in a different stack.
 * consul: a stack realising service discovery for Prometheus and scalelite consiting of the following components
    * lb: a load balancer proving gateway functionality. Should be disabled in production environments.
    * consul: the consul instance. Consule is populated by ansible upon deployment.
 * coturn: a stack provding a coturn server for BBB. 
 * feedback: a stack providing the functionality to collect [BBB user feedback](https://docs.bigbluebutton.org/2.2/customize.html#collect-feedback-from-the-users) 
 * metrics: a stack providing monitoring functionality. This stack consists of the following containers.
   * grafana: a Dashboard for showing monitoring metrics (cpu load, ...)
   * nodeexporter: a container running on each virtual machine and providing system metrics including cpu load, network load, and memory consumption. BBB-specific metrics are captured in the BBB stack.
   * processexporter: a container running on each virtual machine and providing per process system metrics including cpu load, network load, and memory consumption. 
   * prometheus: data sink for the prometheus monitoring
 * postgres: a stack provinding a postgres database management system for greenlight and scalelite.
 * ssh-keys: a stack for applying a set of ssh keys to the virtual machines (overcoming an shortcoming in OpenStack with handling multiple SSH keys). It consists of a single container with `started-once` semantics.

## Our Repository Structure

For our deployment, we try to use as many upstream components unchanged. This, however, is not possible in all cases. In addition, we built several other components to support our deployment:
* apps
   * coturn: repo that packages the coturn server in a Docker container
   * bbb-registrator: a repo implementing the scalelite-registrator container
   * update-ssh-keys: a repo implementing the ssh-keys container
   * rancher-cli: a repo implementing the rancher-cli
* config
   * coturn-certs: Contains the SSL certificates for coturn to enable TLS on STUN and TURN. This is a coturn app sidekick
   * cortun: configuration file including minor templating. This is a coturn app sidekick
   * nginx-feedback: NGINX configuration for feedback collection. This is a sidekick for a generic nginx container
   * update-ssh-keys: place ssh keys for server access here. deploy pipeline via rancher-cli has to be started manual
   * grafana: Provisioning for grafana. Contains datasources and dashboards. This is a sidekick for a generic grafana container.
   * prometheus: Configuration for prometheus. Contains the sevice discovery adapter consuming consul to find its scrape targets. This is a sidekick for a generic prometheus container.
   * processexporter: Configuration for processexporter. Defines a pattern for the processes to be scraped. This is a sidekick for the processexporter app.
* application-single: Vollständige BBB Anwendung inklusive Konfigurationstemplates
* deployment: Vollständiges BBB Deployment auf Basis von Ansible
* tools
   * bbb-feedback-parser: repo with script for parsing bbb feedback logs to csv
* state
   * backup: Daily snapshot of all databases and rancher stacks

## Setting-up your own installation

TBD