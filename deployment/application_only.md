# Application Only

This is a list of all the `docker-compose` files used in this deployment. See the [Run documentation](deployment/run) for variable explanation. For simplicity, `rancher-compose` files are omitted, however rancher labels are still shown to give hints about scheduling.

## bbb-registrator.yml

```yml
 version: '2'
services:
  bbb-registrator:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/apps/bbb-registrator:latest
    environment:
      BBB_SECRET: ${BBB_SECRET}
      CONSUL_HTTP_ADDR: consul.consul:8500
      REDIS_URL: redis://scalelite-redis.bbb-scalelite:6379
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
```

## bbb.yml

```yml
 version: '2'
services:
  bbb:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/application-single:beta4
    privileged: true
    environment:
      - BBB_LOCALIP=${BBB_LOCALIP}
      - BBB_DOMAIN=${BBB_DOMAIN}
      - BBB_HTTP=${BBB_HTTP}
      - BBB_EXTERNALIP=${BBB_EXTERNALIP}
      - BBB_LOCALINTERFACE=${BBB_LOCALINTERFACE}
      - BBB_SECRET=${BBB_SECRET}
      - GREENLIGHT_URI=${GREENLIGHT_URI}
      - BBB_STUN_HOST=${BBB_STUN_HOST}
      - BBB_STUN_PORT=3478
      - BBB_STUN_SECRET=${BBB_STUN_SECRET}
      - BBB_PAD_KEY=${BBB_PAD_KEY}
      - BBB_SIP_ENABLED=${BBB_SIP_ENABLED}
      - BBB_PHONE_USERNAME=${BBB_PHONE_USERNAME}
      - BBB_PHONE_PASSWORD=${BBB_PHONE_PASSWORD}

    tmpfs:
      - /run
      - /run/lock
      - /tmp
    network_mode: host
    labels:
      io.rancher.container.pull_image: always
      io.rancher.container.dns: 'true'
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}

  lb:
    image: rancher/lb-service-haproxy:v0.9.6
    ports:
    - 443:443/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.agent.role: environmentAdmin,agent
      io.rancher.container.agent_service.drain_provider: 'true'
      io.rancher.container.create_agent: 'true'
      io.rancher.scheduler.global: 'true'

  bbb-exporter:
    image: greenstatic/bigbluebutton-exporter:v0.2.0
    ports:
      - "9688:9688"
    environment:
      - API_BASE_URL=http://bbb:8090/bigbluebutton/api/
      - API_SECRET=${API_SECRET}
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
```

## consul.yml

```yml
 version: '2'
services:
  lb:
    image: rancher/lb-service-haproxy:v0.9.6
    ports:
    - 8501:8501/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.agent.role: environmentAdmin,agent
      io.rancher.container.agent_service.drain_provider: 'true'
      io.rancher.container.create_agent: 'true'
  consul:
    image: consul:1.7
    network_mode: host
    command:
    - /bin/consul
    - agent
    - -server
    - -data-dir=/consul/data
    - -bootstrap
    - -client=0.0.0.0
    - -advertise=127.0.0.1
    - -ui
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.dns: 'true'
      io.rancher.container.pull_image: always
```

## coturn.yml

```yml
 version: '2'
services:
  coturn-conf:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/coturn
    labels:
      io.rancher.container.pull_image: always
      io.rancher.container.start_once: 'true'
  coturn-certs:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/coturn-certs
    labels:
      io.rancher.container.start_once: 'true'
      io.rancher.container.pull_image: always
  coturn:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/apps/coturn
    environment:
      TLS_LISTENING_PORT: '3478'
      LISTENING_IP: ${LISTENING_IP}
      EXTERNAL_IP: ${EXTERNAL_IP}
      STATIC_AUTH_SECRET: ${STATIC_AUTH_SECRET}
      SERVER_DOMAIN: ${SERVER_DOMAIN}
      CERT_PATH: ${CERT_PATH}
      KEY_PATH: ${KEY_PATH}
    stdin_open: true
    network_mode: host
    tty: true
    volumes_from:
    - coturn-conf
    - coturn-certs
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.dns: 'true'
      io.rancher.sidekicks: coturn-conf,coturn-certs
      io.rancher.container.pull_image: always
```

## feedback.yml

```yml
 version: '2'
services:
  feedback:
    image: nginx:1.17
    stdin_open: true
    tty: true
    volumes:
    - feedback:/var/log/nginx/
    volumes_from:
    - feedback-config
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.sidekicks: feedback-config
      io.rancher.container.pull_image: always
  feedback-config:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/nginx-feedback:latest
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.pull_image: always
      io.rancher.container.start_once: 'true'
```

## greenlight.yml

```yml
 version: '2'
services:
  lb:
    image: rancher/lb-service-haproxy:v0.9.6
    ports:
    - 443:443/tcp
    - 80:80/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.agent.role: environmentAdmin,agent
      io.rancher.container.agent_service.drain_provider: 'true'
      io.rancher.container.create_agent: 'true'
      io.rancher.scheduler.global: 'true'
  greenlight-mail:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/apps/docker-postfix-relay:latest
    environment:
      MAIL_RELAY_HOST: ${MAIL_RELAY_HOST}
      MAIL_RELAY_PORT: '25'
      MAILNAME: ${MAILNAME}
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always      
  greenlight:
    image: bigbluebutton/greenlight:release-2.5
    environment:
      ALLOW_GREENLIGHT_ACCOUNTS: 'false'
      ALLOW_MAIL_NOTIFICATIONS: 'true'
      BIGBLUEBUTTON_ENDPOINT: ${BIGBLUEBUTTON_ENDPOINT}
      BIGBLUEBUTTON_SECRET: ${BIGBLUEBUTTON_SECRET}
      DB_ADAPTER: postgresql
      DB_HOST: postgresql.postgres
      DB_NAME: greenlight
      DB_PASSWORD: password
      DB_USERNAME: postgres
      DEFAULT_REGISTRATION: invite
      LDAP_BASE: ${LDAP_BASE}
      LDAP_BIND_DN: ${LDAP_BIND_DN}
      LDAP_METHOD: ${LDAP_METHOD}
      LDAP_PASSWORD: ${LDAP_PASSWORD}
      LDAP_PORT: ${LDAP_PORT}
      LDAP_SERVER: ${LDAP_SERVER}
      LDAP_UID: ${LDAP_UID}
      MAINTENANCE_MODE: 'false'
      PORT: '1234'
      RACK_ENV: production
      RAILS_LOG_TO_STDOUT: 'true'
      RELATIVE_URL_ROOT: /
      ROOM_FEATURES: mute-on-join,require-moderator-approval,anyone-can-start,all-join-moderator
      SECRET_KEY_BASE: ${SECRET_KEY_BASE}
      SMTP_DOMAIN: ${SMTP_DOMAIN}
      SMTP_PORT: '25'
      SMTP_SENDER: ${SMTP_SENDER}
      SMTP_SERVER: greenlight-mail.greenlight
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.sidekicks: greenlight-mail
```

## metrics.yml

```yml
 version: '2'
services:
  nodeexporter:
    image: prom/node-exporter:v0.18.1
    volumes:
    - /proc:/host/proc:ro
    - /sys:/host/sys:ro
    - /:/rootfs:ro
    ports:
    - 9100:9100/tcp
    expose:
    - '9100'
    network_mode: host
    command:
    - --path.procfs=/host/proc
    - --path.rootfs=/rootfs
    - --path.sysfs=/host/sys
    - --collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)
    - --collector.processes
    labels:
      io.rancher.container.dns: 'true'
      io.rancher.container.pull_image: always
      io.rancher.scheduler.global: 'true'
  processexporter:
    privileged: true
    image: ncabatoff/process-exporter
    stdin_open: true
    volumes:
    - /proc:/host/proc:ro
    tty: true
    volumes_from:
    - processexporter-config
    ports:
    - 9256:9256/tcp
    expose:
    - '9256'
    command:
    - --procfs
    - /host/proc
    - -config.path
    - /conf/config.yml
    labels:
      io.rancher.container.dns: 'true'
      io.rancher.sidekicks: processexporter-config
      io.rancher.container.pull_image: always
      io.rancher.scheduler.global: 'true'
  grafana:
    image: grafana/grafana:6.7.1
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GF_SECURITY_ADMIN_PASSWORD}
      GF_SECURITY_ADMIN_USER: ${GF_SECURITY_ADMIN_USER}
      GF_USERS_ALLOW_SIGN_UP: 'false'
      GF_ROOT_URL: ${GF_ROOT_URL}
      GF_DOMAIN: ${GF_DOMAIN}
      GF_AUTH_ANONYMOUS_ENABLED: true
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.pull_image: always
      io.rancher.sidekicks: grafana-conf
    volumes_from:
      - grafana-conf
  lb:
    image: rancher/lb-service-haproxy:v0.9.6
    ports:
    - 443:443/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.agent.role: environmentAdmin,agent
      io.rancher.container.agent_service.drain_provider: 'true'
      io.rancher.container.create_agent: 'true'
  prometheus:
    image: prom/prometheus:v2.16.0
    volumes:
    - prometheus_data:/prometheus
    command:
    - --config.file=/conf/prometheus/prometheus.yml
    - --storage.tsdb.path=/prometheus
    - --web.console.libraries=/etc/prometheus/console_libraries
    - --web.console.templates=/etc/prometheus/consoles
    - --storage.tsdb.retention.time=200h
    - --web.enable-lifecycle
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.sidekicks: prometheus-conf
    volumes_from:
      - prometheus-conf
  prometheus-conf:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/prometheus:master
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.start_once: 'true'
      io.rancher.container.pull_image: always
  grafana-conf:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/grafana:latest
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.start_once: 'true'
      io.rancher.container.pull_image: always
    environment:
      GF_AUTH_TOKEN: ${GF_AUTH_TOKEN}
      USERS: ${USERS}
      USER_NAME_1: ${USER_NAME_1}
      USER_LOGIN_1: ${USER_LOGIN_1}
      USER_EMAIL_1: ${USER_EMAIL_1}
      USER_PASSWORD_1: ${USER_PASSWORD_1}
  processexporter-config:
    image: omi-registry.e-technik.uni-ulm.de/kiz/bbb/config/processexporter
    labels:
      io.rancher.container.pull_image: always
      io.rancher.container.start_once: 'true'
```

## postgres.yml

```yml
 version: '2'
services:
  postgresql:
    image: postgres:11.7-alpine
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
```

## scalelite.yml

```yml
 version: '2'
services:
  scalelite-poller:
    image: blindsidenetwks/scalelite:v1-poller
    environment:
      RAILS_LOG_TO_STDOUT: 'true'
      REDIS_URL: redis://scalelite-redis:6379
      INTERVAL: 20
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
  scalelite-api:
    image: blindsidenetwks/scalelite:v1-api
    environment:
      DATABASE_URL: postgresql://postgres:password@postgresql.postgres:5432/scalelite
      LOADBALANCER_SECRET: ${LOADBALANCER_SECRET}
      RAILS_LOG_TO_STDOUT: 'true'
      REDIS_URL: redis://scalelite-redis:6379
      SECRET_KEY_BASE: ${SECRET_KEY_BASE}
      URL_HOST: ${URL_HOST}
    stdin_open: true
    tty: true
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.hostname_override: container_name
      io.rancher.container.pull_image: always
  scalelite-redis:
    image: redis:5.0.8-alpine3.11
    stdin_open: true
    tty: true
    volumes:
      - redis_data:/data
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
  scalelite-registrator:
    image: blindsidenetwks/scalelite:v1-poller
    environment:
      RAILS_ENV: development
      REDIS_URL: redis://scalelite-redis:6379
    stdin_open: true
    tty: true
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
  scalelite-lb:
    image: rancher/lb-service-haproxy:v0.9.6
    ports:
    - 443:443/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: name=${BBB_HOST}
      io.rancher.container.agent.role: environmentAdmin,agent
      io.rancher.container.agent_service.drain_provider: 'true'
      io.rancher.container.create_agent: 'true'
      io.rancher.scheduler.global: 'true'
```
