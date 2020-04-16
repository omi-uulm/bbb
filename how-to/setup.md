# Set-up your own installation

Our installation is based on four steps:

1. acquire resources either from a cloud provider or physical servers
2. install Rancher as container orchestration tool on one of the servers
3. deploy Rancher agents over the remaining servers
4. deploy applications through Rancher

This page describes the installation of Greenlight, BBB and Scalelite on Virtual Machines using the Rancher orchestrator. Principially, indiviudal instances of BBB can be deployed isolated without and orchestrator. Also, complex, distributed deployments can principially be operated without orchestrator; yet, this has not been validated and is not covered in this tutorial.

## Acquire virtual machines 

## Rancher and Rancher agents

TBD

## BBB

For each instance of BBB, you require:

* a host with 
  * a public IP address
  * a DNS entry pointing to that IP address either directly (A record) or indirectly (CNAME record)
  * an SSL certificate for this DNS entry

TBD

## Greenlight

TBD

### Mail Support

* Set environment variable ...

## Scalelite

TBD
