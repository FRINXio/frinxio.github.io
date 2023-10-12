# Uniconfig Properties

## Introduction

UniConfig has input parameters to configure it precisely.
All these parameters are located in the *application.properties* file:

```
# Server settings
server.port=8181
server.servlet.context-path=/rests
server.address=0.0.0.0

# Jetty settings
server.jetty.threads.max=200
server.jetty.threads.min=8
server.jetty.threads.idle-timeout=60s

# TLS settings
#server.http2.enabled=true
#server.ssl.enabled=true
#server.ssl.protocol=TLS
#server.ssl.client-auth=need
#server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
#server.ssl.ciphers=
#server.ssl.trust-store=.truststore
#server.ssl.trust-store-password=trust-pass
#server.ssl.key-store=.keystore
#server.ssl.key-store-password=passtest
# Enable SNI
#server.ssl.sni-enabled=true

# Controller configuration
controller.dom-notification-router-config.queue-depth=65536
controller.dom-notification-router-config.spin-time=1
controller.dom-notification-router-config.park-time=30
controller.dom-notification-router-config.unit=MILLISECONDS

#controller.schema-service-config.top-level-modules[0]=
controller.schema-service-config.root-cache-directory=cache
controller.schema-service-config.system-schemas-directory=schemas
controller.schema-service-config.invalid-files-limit=100
controller.schema-service-config.max-built-yang-repositories=64
controller.schema-service-config.built-yang-repository-expiration=600

controller.configuration-datastore-config.max-data-change-executor-pool-size=20
controller.configuration-datastore-config.max-data-change-executor-queue-size=1000
controller.configuration-datastore-config.max-data-change-listener-queue-size=1000
controller.configuration-datastore-config.max-data-store-executor-queue-size=5000

controller.operational-datastore-config.max-data-change-executor-pool-size=20
controller.operational-datastore-config.max-data-change-executor-queue-size=1000
controller.operational-datastore-config.max-data-change-listener-queue-size=1000
controller.operational-datastore-config.max-data-store-executor-queue-size=5000

# mount-point service configuration
controller.dom-mountpoint-service-config.notification-pool-core-size=-1
controller.dom-mountpoint-service-config.notification-pool-max-size=-1
controller.dom-mountpoint-service-config.notification-queue-capacity=-1
controller.dom-mountpoint-service-config.notification-pool-keep-alive-time=120
controller.dom-mountpoint-service-config.mountpoint-tree-pruning-interval=60


# DB persistence settings
db-persistence.embedded-database.enabled=false
db-persistence.embedded-database.data-dir=./data/pg_dir
db-persistence.embedded-database.clean-data-dir=true

db-persistence.connection.db-name=uniconfig
db-persistence.connection.username=uniremote
db-persistence.connection.password=unipass
db-persistence.connection.uri=jdbc:postgresql://
db-persistence.connection.driver-class-name=org.postgresql.Driver
db-persistence.connection.connection-timeout=30000
db-persistence.connection.max-lifetime=1800000
db-persistence.connection.min-idle-connections=10
db-persistence.connection.max-db-pool-size=20
db-persistence.connection.socket-read-timeout=20
db-persistence.connection.enabled-tls=false
db-persistence.connection.tls-client-cert=./client.pks
db-persistence.connection.tls-client-key=./client.key
db-persistence.connection.tls-ca-cert=./ca.pks
db-persistence.connection.ssl-password=
db-persistence.connection.database-locations[0].host=127.0.0.1
db-persistence.connection.database-locations[0].port=26257
db-persistence.connection.repair-schema-history=false
#db-persistence.uniconfig-instance.instance-name=
db-persistence.uniconfig-instance.host=127.0.0.1


# Uniconfig cloud settings - depends on kafka connection
uniconfig.cloud.config.enabled=true
uniconfig.cloud.config.fail-fast=false
uniconfig.cloud.config.application-name=uniconfig
spring.cloud.bus.enabled=true
spring.kafka.bootstrap-servers=http://localhost:9092
management.endpoints.web.exposure.include=bus-refresh
# Disable cloud connection to Kafka for testing purpose, property spring.cloud.bus.enabled must be false
#spring.autoconfigure.exclude=org.springframework.cloud.stream.function.FunctionConfiguration


# Template settings
templates.enabled=false
templates.latest-schema-reference-module-name=system
templates.enabled-templates-upgrading=false
templates.backup-templates-limit=7


# RESTCONF settings
restconf.showNamespaceInJsonResponse=true
restconf.formatErrorWithLatestSchema=false
restconf.yangPatchRfc8040ErrorResponse=false
restconf.status-code-for-empty-get-response=0
restconf.schema-filters.ignore-unsupported-definitions-on-write=false
restconf.schema-filters.hide-deprecated-definitions-on-read=false
restconf.schema-filters.ignored-data-on-write-by-extensions[0]=tailf:hidden full
restconf.schema-filters.hidden-data-on-read-by-extensions[0]=tailf:hidden deprecated
restconf.schema-filters.hidden-data-on-read-by-extensions[1]=tailf:hidden debug
#restconf.key-delimiter=


# CLI shell config
cli-shell.default-unistore-node-id=system
cli-shell.default-callbacks-repository=callbacks
cli-shell.enable-scrolling=false
cli-shell.history-size=500
cli-shell.history-file-size=1000
cli-shell.ssh-server.enabled=false
cli-shell.ssh-server.port=2022
cli-shell.ssh-server.inet-address=127.0.0.1
cli-shell.ssh-server.username-password-auth.username=admin
cli-shell.ssh-server.username-password-auth.password=secret
cli-shell.shell-colors.description-color=default
cli-shell.shell-colors.error-color=default
cli-shell.shell-colors.info-color=default
cli-shell.shell-colors.warning-color=default


# Settings for the Device Discovery service
device-discovery.local-address=
device-discovery.initial-pool-size=1
device-discovery.max-pool-size=20
device-discovery.keepalive-time=60
device-discovery.address-check-limit=254


# Grouped settings that are related to Uniconfig transactions
transactions.transaction-idle-time-out=300
transactions.max-transaction-age=1800
transactions.cleaning-interval=60
transactions.immediate-commit-enabled=true


# Grouped settings that are related to Task executor
task-executor.max-queue-capacity=10000
task-executor.max-cpu-load=0.9
task-executor.default-thread-count=
task-executor.max-thread-count=
task-executor.keepalive-time=60


# Grouped settings that are related to notifications.
notifications.enabled=false
notifications.max-subscriptions-hard-limit=5000
notifications.max-subscriptions-per-interval=10
notifications.subscriptions-monitoring-interval=5
notifications.audit-logs.include-response-body=false
notifications.audit-logs.include-calculate-diff-result=false
notifications.notification-db-threshold.max-count=10000
notifications.notification-db-threshold.max-age=100

#thread-parameters.monitoring-executor-initial-pool-size=
#thread-parameters.monitoring-executor-maximum-pool-size=
#thread-parameters.monitoring-executor-keepalive-time=60


# Uniconfig settings
uniconfig.autogenerated_config_dir=config


logging.config=config/logback.xml


# Metrics config
metrics.enabled=true
metrics.reporterType=LOG
metrics.rate=30
# example csv output for one metric
#        t,count,mean_rate,m1_rate,m5_rate,m15_rate,rate_unit
#        1650557436,3,0.182204,0.507889,0.580330,0.593370,events/second
#        1650557466,3,0.064564,0.308050,0.525104,0.573917,events/second
#        1650557496,3,0.039233,0.186842,0.475134,0.555102,events/second
#        1650557526,3,0.028178,0.113325,0.429919,0.536904,events/second
#        1650557534,3,0.026281,0.104264,0.422813,0.533929,events/second
#
# example log output
# 18:16:20.886 INFO io.frinx.uniconfig.metrics.impl.ReporterImpl - type=GAUGE, name=io.frinx.uniconfig.manager.impl.task.TaskExecutorImpl.queue_size, value=0
# 18:16:20.887 INFO io.frinx.uniconfig.metrics.impl.ReporterImpl - type=METER, name=org.opendaylight.controller.uniconfig.transaction.manager.impl.UniconfigTransactionManagerImpl.transaction_invoke, count=0, m1_rate=0.0, m5_rate=0.0, m15_rate=0.0, mean_rate=0.0, rate_unit=events/second
# 18:16:20.887 INFO io.frinx.uniconfig.metrics.impl.ReporterImpl - type=METER, name=org.opendaylight.yangtools.yang.common.RpcResult.rpc_invoke, count=3, m1_rate=0.3080502714195554, m5_rate=0.5251039914257685, m15_rate=0.5739172434618175, mean_rate=0.06565517632771098, rate_unit=events/second


connection-manager.amount-of-max-parallel-installs=20

sync-from-network.partition-size=0
sync-from-network.partition-read-timeout=5

heart-beat.heart-beat-interval=1000


# Netconf topology parameters
netconf-topology-parameters.fixed-thread-pool-thread-count=2
netconf-topology-parameters.scheduled-thread-pool-thread-count=2


# SSH Client properties
ssh-client.default-timeout=-1
ssh-client.heartbeat-interval=30
ssh-client.heartbeat-reply-wait=120
ssh-client.heartbeat-request=keepalive@sshd.apache.org
ssh-client.ssh-default-nio-workers=8


# CLI topology parameters
cli-topology-parameters.keepalive-thread-count=
cli-topology-parameters.init-executor-thread-timeout=120
cli-topology-parameters.init-executor-thread-count=


# NETCONF setup
netconf.topology-id=topology-netconf
netconf.write-tx-timeout=0


# Logging controller config
#   Logging all NETCONF messages.
logging-controller.brokers[0].broker-identifier=netconf_messages
#logging-controller.brokers[0].device-list[0]=dev1
logging-controller.brokers[0].logging-broker-enabled=false
logging-controller.brokers[0].logging-enabled-on-all-devices=false

#   Logging all southbound NETCONF events related to creation/closing of NETCONF connections.
logging-controller.brokers[1].broker-identifier=netconf_events
logging-controller.brokers[1].logging-broker-enabled=false
logging-controller.brokers[1].logging-enabled-on-all-devices=false

#   Logging all NETCONF notification messages.
logging-controller.brokers[2].broker-identifier=netconf_notifications
logging-controller.brokers[2].logging-broker-enabled=false
logging-controller.brokers[2].logging-enabled-on-all-devices=false

#   Logging all CLI commands and responses.
logging-controller.brokers[3].broker-identifier=cli_messages
logging-controller.brokers[3].logging-broker-enabled=false
logging-controller.brokers[3].logging-enabled-on-all-devices=false

#   Logging RESTCONF requests and responses.
logging-controller.brokers[4].broker-identifier=restconf
logging-controller.brokers[4].logging-broker-enabled=false
#logging-controller.brokers[4].restconf-logging-hidden-http-headers[0]=Authorization
#logging-controller.brokers[4].restconf-logging-hidden-http-headers[1]=Cookie
#logging-controller.brokers[4].restconf-logging-hidden-http-methods[0]=GET
#        HTTP headers which content is removed from logs.
#       "restconf-logging:hidden-http-headers": [
#           "Authorization",
#          "Cookie"
#       ],
#        HTTP operations that are not logged.
#       "restconf-logging:hidden-http-methods": [
#          "GET"
#       ]

#   Logging Uniconfig Shell commands and responses.
logging-controller.brokers[5].broker-identifier=shell_logs
logging-controller.brokers[5].logging-broker-enabled=false

#   Logging all gNMI messages.
logging-controller.brokers[6].broker-identifier=gnmi_messages
logging-controller.brokers[6].logging-broker-enabled=false
logging-controller.brokers[6].logging-enabled-on-all-devices=false
#logging-controller.brokers[6].device-list[0]=dev1

# southbound RESTCONF connection settings used by UniConfig shell
restconf.southbound.request-base-uri=http://localhost:8181/rests
restconf.southbound.request-timeout=60
restconf.southbound.connect-timeout=60


# MUTABLE PROPERTIES - SAVED IN DATABASE AND CAN BE CHANGED BY RPC:

# Cryptographic settings
# Update encryption status rpc works only with notifications.enabled=true
crypto.encrypt-enabled=true
crypto.encryption-rotation-batch-size=10
crypto.encrypt-extension-id=frinx-encrypt:encrypt
crypto.netconf-reference-module-name=system
crypto.netconf-encrypted-paths-module-name=encrypted-paths
crypto.new-encryption-cipher-type=
crypto.new-encryption-public-key=
crypto.new-decryption-private-key=


schema-settings.latest-schema=
schema-settings.latest-reference-module-revision=


# Callbacks config
callbacks.enabled=false
callbacks.remote-server.host=
callbacks.remote-server.port=


# Grouped settings that are related to kafka.
notifications.kafka.audit-logs-enabled=true
notifications.kafka.netconf-notifications-enabled=true
notifications.kafka.gnmi-notifications-enabled=true
notifications.kafka.snmp-notifications-enabled=true
notifications.kafka.snmp-notifications-udp-port=50000
notifications.kafka.snmp-notifications-address-cache-duration=600
notifications.kafka.snmp-notifications-oid-cache-duration=600
notifications.kafka.transaction-notifications-enabled=true
notifications.kafka.data-change-events-enabled=true
notifications.kafka.connection-notifications-enabled=true
notifications.kafka.netconf-notifications-topic-name=netconf-notifications
notifications.kafka.gnmi-notifications-topic-name=gnmi-notifications
notifications.kafka.snmp-notifications-topic-name=snmp-notifications
notifications.kafka.audit-logs-topic-name=audit-logs
notifications.kafka.transactions-topic-name=transactions
notifications.kafka.data-change-events-topic-name=data-change-events
notifications.kafka.connection-notifications-topic-name=connection-notifications
notifications.kafka.connection-notifications-netconf-stream-only=true
notifications.kafka.max-thread-pool-size=8
notifications.kafka.queue-capacity=2048
notifications.kafka.shell-notifications-topic-name=shell-notifications
notifications.kafka.shell-notifications-enabled=true
notifications.kafka.kafka-servers[0].broker-host=127.0.0.1
notifications.kafka.kafka-servers[0].broker-listening-port=9092
notifications.kafka.kafka-producer.batch-size=16384
notifications.kafka.kafka-producer.request-timeout=30000
notifications.kafka.kafka-producer.delivery-timeout=120000
notifications.kafka.kafka-producer.blocking-timeout=60000
notifications.kafka.kafka-producer.linger=0
notifications.kafka.kafka-subscriber.netconf-notifications-poll-timeout=100
notifications.kafka.kafka-subscriber.data-change-events-poll-timeout=100
notifications.kafka.embedded-kafka.enabled=false
notifications.kafka.embedded-kafka.install-dir=/tmp/embedded-kafka
notifications.kafka.embedded-kafka.archive-url=https://dlcdn.apache.org/kafka/3.4.1/kafka_2.13-3.4.1.tgz
notifications.kafka.embedded-kafka.data-dir=./data/embedded-kafka
notifications.kafka.embedded-kafka.clean-data-before-start=true
notifications.kafka.embedded-kafka.partitions=1

## kafka sasl
#notifications.kafka.username=
#notifications.kafka.password=

## or kafka ssl
#notifications.kafka.ssl-keystore-location=
#notifications.kafka.ssl-keystore-password=
#notifications.kafka.ssl-truststore-location=
#notifications.kafka.ssl-truststore-password=
#notifications.kafka.ssl-key-password=


# Netconf default parameters properties.
netconf-default-parameters.flags.enabled-notifications=true
netconf-default-parameters.flags.enabled-strict-parsing=true
netconf-default-parameters.flags.reconnect-on-changed-schema=false
netconf-default-parameters.flags.streaming-session=false

netconf-default-parameters.session-timers.between-attempts-timeout=2000
netconf-default-parameters.session-timers.confirm-commit-timeout=600
netconf-default-parameters.session-timers.initial-connection-timeout=20000
netconf-default-parameters.session-timers.keepalive-delay=120
netconf-default-parameters.session-timers.max-connection-attempts=1
netconf-default-parameters.session-timers.max-reconnection-attempts=0
netconf-default-parameters.session-timers.reconnection-attempts-multiplier=1.5
netconf-default-parameters.session-timers.request-transaction-timeout=60000

netconf-default-parameters.other-parameters.concurrent-rpc-limit=0
netconf-default-parameters.other-parameters.dry-run-journal-size=0
netconf-default-parameters.other-parameters.custom-connector-factory=default
netconf-default-parameters.other-parameters.edit-config-test-option=test-then-set


# GNMI default parameters
gnmi-default-parameters.session-timers.request-timeout=30
gnmi-default-parameters.session-timers.internal-transaction-timeout=30
gnmi-default-parameters.session-timers.request-max-size=4194304
gnmi-default-parameters.flags.enabled-notifications=true
gnmi-default-parameters.other-parameters.dry-run-journal-size=0


# CLI default parameters
cli-default-parameters.max-connection-attempts=1
cli-default-parameters.max-reconnection-attempts=0
cli-default-parameters.max-connection-attempts-install=1
cli-default-parameters.dry-run-journal-size=0
cli-default-parameters.journal-level=command-only
cli-default-parameters.journal-size=0
cli-default-parameters.keepalive-delay=60
cli-default-parameters.keepalive-initial-delay=120
cli-default-parameters.keepalive-timeout=60
cli-default-parameters.command-timeout=60
cli-default-parameters.connection-establish-timeout=60
cli-default-parameters.connection-lazy-timeout=60
cli-default-parameters.parsing-engine=tree-parser
```

These parameters are divided into 2 groups - immutable parameters, 
which are the majority, and mutable parameters, which can be
changed in runtime without restarting UniConfig with *UniConfig Cloud Config*.

Mutable properties consist of *crypto*, *schema-settings*, *callbacks*,
*notifications.kafka*, *netconf-default-parameters*, *gnmi-default-parameters*
and *cli-default-parameters* property prefixes. These properties are stored
in the database in the table *properties* and are known as *default properties*. 

!!!
So after starting UniConfig, if the database contains these properties - 
UniConfig will use the values of these properties from the database. But if these 
parameters are not in the database, then the values from first UniConfig instance 
right after its startup will be used (by application.properties file or env variables) 
and saved to the database for the next UniConfig instances.
!!!

## UniConfig Cloud Config

UniConfig Cloud Config is pretty the same technology as Spring Cloud Config with JDBC backend:
https://docs.spring.io/spring-cloud-config/reference/4.1/server/environment-repository/jdbc-backend.html 
and Spring Cloud Bus: https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/.

The main difference between the two is that Uniconfig Cloud Config Server and 
Cloud Config Client are in the same project, while Spring requires the use of a separate
Cloud Config Server application.

It works as follows:

* First, before starting UniConfig, we must enable Cloud Config using the following properties

```
# Uniconfig cloud settings - depends on kafka connection
uniconfig.cloud.config.enabled=true
uniconfig.cloud.config.fail-fast=false
uniconfig.cloud.config.application-name=uniconfig
spring.cloud.bus.enabled=true
spring.kafka.bootstrap-servers=http://localhost:9092
management.endpoints.web.exposure.include=bus-refresh
# Disable cloud connection to Kafka for testing purpose, property spring.cloud.bus.enabled must be false
#spring.autoconfigure.exclude=org.springframework.cloud.stream.function.FunctionConfiguration
```

* When UniConfig starts, it will first check if there are any *default properties* in the database that it could configure
  * If *default properties* are in the database -> UniConfig instance will use it
  * If not -> UniConfig instance will use its properties and, once loaded, will save them to a database 
    for a following UniConfig instances.
* At the end of Spring initialisation the *Refresh Bus Endpoint* will be called. Its refresh *default properties* 
  to the database values in all UniConfig instances connected via kafka refresh topic.
* At application runtime, if user uses *update-properties RPC* with *default properties* on input -> UniConfig will
  update the properties in database and then use the *Refresh Bus Endpoint* call, which will reload properties in all
  connected via kafka UniConfig instances.

```




                                                      +---------------------+
                                                      | Kafka               |
                                                      |                     |
                                                      |                     |
                                                      +----------^----------+             STEP 2:
                                                                 |                        Changing RPCs sends changes to database and
                                                                 |                        sends notification to kafka about 
                                                                 |                        configuration change. Other UC instances receive 
                                                                 |                        this notification and execute bus-refresh call
                                           +---------------------+--------------------+   to reload configuration properties (from Postgres)
                                           |                                          |
                                           |                                          |
    STEP 1:                        +-------+-------+                          +-------+-------+
    Uniconfig user                 | Uniconfig 1   |                          | Uniconfig 2   |
    modifies config  +------------->               |                          |               |
    properties via                 |               |                          |               |
    REST api                       +-------+-------+                          +-------+-------+
                                           |                                          |
                                           |                                          |
                                           +---------------------+--------------------+   STEP 3:
                                                                 |                        Uniconfig cloud config module
                                                                 |                        reads new config properties from
                                                                 |                        Postgres properties table and
                                                                 |                        reloads new properties to UC instance
                                                         +-------v-------+
                                                         | Postgres      |
                                                         |               |
                                                         |               |
                                                         +---------------+

```

How to disable UniConfig Cloud Config and how the properties will be set up:

* First, before starting UniConfig, we must disable Cloud Config using the following properties

```
# Uniconfig cloud settings - depends on kafka connection
uniconfig.cloud.config.enabled=false
uniconfig.cloud.config.fail-fast=false
uniconfig.cloud.config.application-name=uniconfig
spring.cloud.bus.enabled=false
spring.kafka.bootstrap-servers=http://localhost:9092
management.endpoints.web.exposure.include=bus-refresh
# Disable cloud connection to Kafka for testing purpose, property spring.cloud.bus.enabled must be false
spring.autoconfigure.exclude=org.springframework.cloud.stream.function.FunctionConfiguration
```

* When UniConfig starts, it will first check if there are any *default properties* in the database that it could configure
  * If *default properties* are in the database -> UniConfig instance will use it
  * If not -> UniConfig instance will use its properties and, once loaded, will save them to a database
  for a following UniConfig instances.
* At the end of Spring initialisation the *Refresh Bus Endpoint* wouldn't be called.
* At application runtime, if user uses *update-properties RPC* with *default properties* on input -> UniConfig will
  update the properties in database, but wouldn't be update properties inside application, so it would have affect only
  for next UniConfig instance, which will be start after properties update.
