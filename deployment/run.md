# Run

## Download

You can clone the necessary source files here: <https://omi-gitlab.e-technik.uni-ulm.de/kiz/bbb/deployment-release>

## Requirements

* `docker`
* `docker-compose`
* `make`

## Configuration

### Environment Variables

`BW_OS_USERNAME` and `BW_OS_PASSWORD` have to set. They can either be exported on your current shell, or set in the (not versioned) `.env` file in this repositories root directory. Example

```bash
BW_OS_USERNAME=MY_BW_OS_USERNAME
BW_OS_PASSWORD=MY_BW_OS_PASSWORD
```

These are your OpenStack login credentials

### Public/Private Key

Supply a public and private keypair in `infrastructure/key` and `infrastructure/key.pub`. This key is being deployed upon instance creation and can be used to access the VM via SSH. It is also used by ansible to bootstrap `rancher` and `consul`

### Certificates

Each server/service has to have a valid SSL certificate. For example:

* `bbb-01.bbb.example.com`
* `bbb-02.bbb.example.com`
* `bbb-03.bbb.example.com`
* `bbb-04.bbb.example.com`
* `bbb-05.bbb.example.com`
* `bbb-06.bbb.example.com`
* `bbb-07.bbb.example.com`
* `bbb-08.bbb.example.com`
* `bbb-09.bbb.example.com`
* `bbb-10.bbb.example.com`
* `bbb.example.com`
* `monitoring.bbb.example.com`
* `rancher.bbb.example.com`
* `scalelite.bbb.example.com`

Place the corresponding `.key` and `.pem` files into the `infrastructure/certs/$CLOUD` directory. Make sure to also place a file named `chain` into that folder, to correctly resolve the certificate chain.

### ansible vars

Create vars files in `infrastructure/playbooks/vars/${CLOUD}.yml` and `infrastructure/playbooks/vars/${CLOUD}_secrets.yml`

```yaml
# infrastructure/playbooks/vars/${CLOUD}.yml
cloud: #cloudname
use_private_net: #true/false
global_network: # openstack network name
global_image: # openstack image name - should be coreos/flatcar
global_flavor: # openstack image flavor
rancher_username: # username for rancher orchestrator
greenlight_uri: # uri for greenlight: "https://bbb.example.com"
greenlight_host: # host for greenlight: "bbb.example.com"
stun_host: # STUN/TURN host: "scalelite.bbb.example.com"
scalelite_uri: # scalelite uri: "https://scalelite.bbb.example.com/"
scalelite_host: # scalelite host: "scalelite.bbb.example.com"
bbb_host: # bbb host: "bbb.example.com"
ldap_base: # "ou=people,dc=example,dc=com"
ldap_bind_dn: # "cn=admin,ou=people,ou=admin,dc=example,dc=com"
ldap_method: # "ssl"
ldap_port: # "'636'"
ldap_server: # "ldap.example.com"
ldap_uid: # "uid"
smtp_host: # "example.com"
smtp_sender: # "noreply@example.com"
grafana_host: # "monitoring.bbb.example.com"
grafana_uri: # "https://monitoring.bbb.example.com"
grafana_username: # username for grafana dashboard
coturn_cert_path: # "/certs/scalelite.bbb.example.com.pem"
coturn_key_path: # "/certs/scalelite.bbb.example.com.key"
mail_relay_host: # "smtp.example.com"
mailname: # "bbb.example.com"
docker_registry_username: # username to login into private registry
```

```yaml
# infrastructure/playbooks/vars/${CLOUD}_secrets.yml
rancher_password: #
bbb_secret: #
stun_secret: #
pad_secret: #
scalelite_secret: #
scalelite_secret_base: #
greenlight_secret_base: #
ldap_password: #
grafana_password: #
docker_registry_password: #
sip_configuration:
  - { enabled : "true",  number: "xxx", password: "xxx" }
  - { enabled : "false", number: "xxx", password: "xxx" }
```

## Running

```bash
make bwcloud-infrastructure
```
