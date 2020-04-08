# Process

## Overview

Broadly speaking, the deployment process is divided into three parts. Everything is handled by Ansible:

1. Using an IaaS provider (in this case OpenStack) to provide the virtual infrastructure
   * See Security settings in OpenStack
   * See Server bootstrapping in OpenStack
2. Bootstrapping a container orchestrator (in this case Rancher1.6)
   * See Server bootstrapping in OpenStack -> `role_server_rancher`
3. Leveraging this container orchestrator to deploy the actual application
   * See Application bootstrapping in Rancher

## Security settings in OpenStack

Fine grained security groups for each service and server are created to only allow access to specific ports. You should supply a private and public key named `infrastructure/key` and `infrastructure/key.pub`, to upload it to OpenStack. This key (public) will be deployed to each VM using cloud-init (CoreOS ignition).

If a private network and floating IPs shall be used, it is handled by the network component.

```mermaid
graph TD;
  %% Comments
  /-->sec_rancher
  sec_rancher --> comment_security_groups>Creates security groups <br> Open Firewall for services]
  sec_bbb --> comment_security_groups
  sec_greenlight --> comment_security_groups
  sec_scalelite --> comment_security_groups
  sec_coturn --> comment_security_groups
  sec_monitoring --> comment_security_groups

  key --> comment_key>Creates public key to <br> enable VM ssh login]

  net --> comment_net>Creates a private network if enabled <br> Disabled for bwCloud]
  sec_rancher --> sec_bbb --> sec_greenlight --> sec_scalelite --> sec_coturn --> sec_monitoring --> key --> net --> server_rancher[Server Setup]
```

## Server bootstrapping in OpenStack

To consequently apply the immutable infrastructure concept we make use of CoreOS and Ignition. Upon first boot, CoreOS applies configuration received from ignition to configure the operating system. This includes, files, services, network configuration, mounts and possibly more.

The rancher server is a special case, since the via ignition bootstrapped application "Rancher" has to be configured. This is done by applying the rancher role, which uses curl and the Rancher API to apply this configuration. Here the following items are configured:

* Environment
* Docker registry
* Access credentials
* Registration token

These configuration options are locally cached and should be part of version control. They are later consumed by the other servers to register to the rancher server. They are also consumed by the application deployment to access the Rancher API.

```mermaid
graph LR;
  ignition_server_rancher{{ignition/rancher}}-->server_rancher
  role_server_rancher{{role_rancher}}-->server_rancher
  comment_role_server_rancher>Setup and configure rancher<br>server software and access credentials] --> role_server_rancher

  %%server_rancher --> comment_server>Server Bootstrapping]
  %%server_bbb --> comment_server
  %%server_greenlight --> comment_server
  %%server_scalelite --> comment_server
  %%server_monitoring --> comment_server
  state_server[(Local Server Metadata<br>Facts)]
  server_rancher --> state_server
  server_bbb --> state_server
  server_greenlight --> state_server
  server_scalelite --> state_server
  server_monitoring --> state_serve
  
  ignition_server_bbb{{ignition/bbb}}-->server_bbb
  ignition_server_greenlight{{ignition/greenlight}}-->server_greenlight  
  ignition_server_scalelite{{ignition/scalelite}}-->server_scalelite
  ignition_server_monitoring{{ignition/monitoring}}-->server_monitoring

    previous[Security settings in OpenStack] --> server_rancher --> server_bbb --> server_greenlight --> server_scalelite --> server_monitoring -->application_bwcloud
```

## Application bootrapping in Rancher

The application deployment simply applies all application descriptions in `infrastructure/apps`. Every application consists of a `*.yml` and a corresponding `*-rancher.yml` file. Some services are registered to consul to make use of prometheus' service discovery and to implement a custom service discovery for all bbb instances.

```mermaid
graph LR;
    state_server[(Local Server Metadata<br>Facts)]
    state_consul[(Remote Consul Metadata<br>Services)]

    state_server --> application_bwcloud
    state_server --> feedback
    state_server --> bbb
    state_server --> greenlight
    state_server --> scalelite
    state_server --> metrics
    state_server --> consul
    state_server --> registrator
    state_server --> postgres
    state_server --> coturn
    state_consul --> registrator
    state_consul --> metrics
    %%state_server --> application_bwcloud

    application_bwcloud --> feedback --> bbb --> greenlight --> scalelite --> metrics --> consul --> registrator --> postgres --> coturn
```

## Application Architecture

```mermaid
graph LR;
    subgraph Feedback
      service_feedback-->|configured by|service_feedback-config
      service_feedback-config
    end
    subgraph coturn
      service_coturn-conf
      service_coturn-certs
      service_coturn-->|configured by|service_coturn-conf
      service_coturn-->|configured by|service_coturn-certs
    end
    subgraph BBB * X
      service_bbb-->|STUN/TURN lookup|service_coturn
      service_bbb_lb-->|ingress|service_bbb
      service_bbb_lb-->|redirects feedback to|service_feedback
      service_bbb-exporter-->|queries|service_bbb
    end
    subgraph scalelight
      service_scalelite-poller
      service_scalelite-api-->|redirects to|service_bbb_lb
      service_scalelite-redis
      service_scalelite-registrator
      service_scalelite-lb-->|ingress|service_scalelite-api
    end
    subgraph consul
      service_consul
      service_consul_lb-->|ingress|service_consul
    end
    subgraph registrator
      service_bbb-registrator-->|registers bbb instances|service_scalelite-api
      service_bbb-registrator-->|queries|service_consul
    end
    subgraph postgres
      service_postgres
    end
    subgraph greenlight
      service_greenlight-->|redirect to|service_scalelite-lb
      service_greenlight-->|manages user db|service_postgres
      service_greenlight-lb-->|ingress|service_greenlight
    end
    subgraph monitoring
      service_monitoring_lb-->|ingress|service_grafana
      service_monitoring_lb-->|ingress|service_prometheus
      service_nodeexporter
      service_processexporter-->|configured by|service_processexporter-config
      service_grafana-->|configured by|service_grafana-conf
      service_grafana-conf
      service_prometheus-->|configured by|service_prometheus-conf
      service_prometheus-->|queries|service_processexporter
      service_prometheus-->|queries|service_nodeexporter
      service_prometheus-->|queries|service_bbb-exporter
      service_prometheus-conf
      service_processexporter-config
    end

```

## Manual Steps

### Register the Rancher server in its own orchestrator

Out of convenience, the rancher Server itself is not registered within any of its environments. To do so a manual step is necessary. For example

```bash
docker run --rm --privileged -e CATTLE_HOST_LABELS='io.rancher.scheduler.require_any=io.rancher.container.system, type=admin&type=admin&name=rancher.novalocal' -e CATTLE_AGENT_IP="$IP" -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.10 $TOKEN
```