# UniConfig Clustering

## Introduction

UniConfig can be easily deployed in the cluster thanks to its stateless architecture and transaction isolation:

* stateless architecture - UniConfig nodes in the cluster don't keep any state that would have to be communicated directly
  to other Uniconfig nodes in a cluster. All network-topology configuration and state information are stored inside
  a PostgreSQL database that must be reachable from all UniConfig nodes in the same zone. All Uniconfig nodes share the
  same database, making the database single source of truth for Uniconfig cluster.
* transaction isolation - Load-balancing is based on mapping UniConfig transactions to Uniconfig nodes
  in a cluster (transactions are sticky). One UniConfig transaction cannot span multiple UniConfig nodes in a cluster.
  Southbound sessions used for device management are ephemeral - they are created when UniConfig needs to access device
  on the network (like pushing cnfiguration updates) and they are closed as soon as a UniConfig transactions is committed or closed.

There are several advantages of clustered deployment of UniConfig nodes:

* horizontal scalability - Increasing number of units that can process UniConfig transactions in parallel.
  Single UniConfig node tends to have limited processing and networking resources - by increasing number of nodes in the
  cluster, this constraint can be mitigated. The more Uniconfig nodes in a cluster, the more transactions can be executed in parallel.
  Number of connected UniConfig nodes in the cluster can also be adjusted at the runtime.
* high-availability - Single UniConfig node doesn't represent single point of failure. If UniConfig node crashes,
  only UniConfig transactions that are processed by corresponding node, are cancelled. Application can retry failed
  transaction, and it will be processed by next node in the cluster.

There also are a couple limitations to be considered:

* Parallel execution of transactions is subject to a locking mechanism, where 2 transactions cannot manipulate the same device at the same time.
* Single transaction is always executed by a single Uniconfig node. This means that a scope of a single transaction is limited by the number devices and   their configuration a single Uniconfig node can handle.

## Deployments

### Single-zone deployment

In the single-zone deployment, all managed network devices are reachable by all UniConfig nodes in the cluster - zone.
Components of the single-zone deployment and connections between them are depicted by the next diagram.

![Deployment with single zone](single-zone-architecture.svg)

Description of components:
* UniConfig controllers - Network controllers that use common PostgreSQL system for persistence of data, communicate
  with network devices using NETCONF/GNMi/CLI management protocols and propagate notifications into Kafka topics
  (UniConfig nodes act only as Kafka producers). UniConfig nodes do not communicate with each other directly,
  their operation can only be coordinated using data stored in the database.
* Database storage - PostgreSQL is used for persistence of network-topology configuration, mountpoints settings,
  and selected operational data. PostgreSQL database can also be deployed in the cluster (outside of scope).
* Message and notification channels - Kafka cluster is used for propagation of notifications that are generated
  by UniConfig itself (e.g., audit and transaction notifications) or from network devices and only propagated
  by UniConfig controller.
* Load-balancers - Load-balancer is used for distributing transactions (HTTP traffic) and SSH sessions
  from applications to UniConfig nodes. From the view of load-balancer, all UniConfig nodes in a cluster are equall.
  Currently, only round-robin load-balancing strategy is supported.
* Managed network devices - Devices that are managed using NETCONF/GNMi/CLI protocols by UniConfig nodes or generate
  notifications to UniConfig nodes. Sessions between UniConfig nodes and devices are either on-demand/emphemeral
  (configuration of devices) or long-term (distribution of notifications over streams).
* HTTP / SSH clients & Kafka consumers - Application layer such as workflow management systems or end-user systems.
  RESTCONF API is exposed using HTTP protocol, SSH server is exposing UniConfig shell
  and Kafka brokers allow Kafka consumers to listen to the events on subscribed topics.

### Multi-zone deployment

In this type of deployment there are multiple zones that manage separate sets of devices because:

* network reachability issues - groups of devices are only reachable and thus manageable from some part
  of the network (zone) but not from others
* logical separation - there are different scaling strategies or requirements for different zones
* legal issues - some devices must be managed separately with split storage, for example, because of the regional
   restrictions

The following diagrams represents a sample deployment with 2 zones. The first zone contains 3 UniConfig nodes
while the second zone contains only 2 UniConfig nodes.
Multiple zones might share a single Kafka cluster but database instances need to be split (could be running in a single postgres server).

![Deployment with multiple zones](multi-zone-architecture.svg)

Description of multi-zone areas:

* Applications - The application layer is responsible for managing mapping between network segments and Uniconfig zones.
  Typically this is achieved by deploying/using an additional inventory database that contains device <-> zone
  mappings - based on this information the application decides which zone to use. 
* Isolated zones - A zone contains one or more UniConfig nodes, load-balancers and managed network devices.
  The clusters in isolated zones share 0 information.
* PostgreSQL databases - It is necessary to use dedicated database per zone.
* Kafka cluster - Kafka cluster can be shared by multiple clusters in different zones or there could be single Kafka cluster per zone.
  Notifications from different zones can be safely pushed to the common topics since there can be no possible
  conflicts between Kafka publishers. However it is also possible to achieve isolation of published messages
  in a shared Kafka deployment by setting different topic names in different zones.

### Load-balancer operation

The responsibility of a load-balancer is to allocate UniConfig transaction on one of the UniConfig nodes in the cluster.
It is done by forwarding requests without UniConfig transaction header to one of the UniConfig nodes
(using round-robin strategy) and afterwards appending a backed identifier to the create-transaction RPC response in form
of an additional Cookie header ('sticky session' concept). Afterwards, it is the responsibility of an application
to assure that all requests that belong to the same transaction contain the same backend identifier.

!!!
The application is responsible for preserving transaction and backend identifier cookies throught a transaction lifetime.
!!!

The next sequence diagram captures a process of creating and using 2 UniConfig transactions with focus
on load-balancer operation.

![Load-balancing UniConfig transactions](load-balancing-transactions.svg)

* The first create-transaction RPC is forwarded to the first UniConfig node (applying round-robin strategy), because
  it does not contain uniconfig_server_id key in the Cookie header.
  The response contains both UniConfig transaction ID (UNICONFIGTXID) and uniconfig_server_id that represents
  'sticky cookie'. Cookie header uniconfig_server_id is appended to the response by load-balancer.
* The next request that belongs to the created transaction, contains same UNICONFIGTXID and uniconfig_server_id.
  Load balancer uses the uniconfig_server_id to forward this request to the correct UniConfig node.
* The last application request represents again create-transaction RPC. This time, request is forwarded to
  the next registered UniConfig node in the cluster according to the round-robin strategy.

## Configuration

### UniConfig configuration

All UniConfig nodes in the cluster should be configured with the same parameters. There are several important sections
of config/application.properties file related to clustered environment.

#### Database connection settings

This section contains information how to connect to PostgreSQL database and connection pool settings. It is placed
under 'dbPersistence.connection' properties object.

Example with essential settings:

```properties database connection settings
# DB persistence settings
db-persistence.embedded-database.enabled=true
db-persistence.embedded-database.data-dir=./data/pg_dir
db-persistence.embedded-database.clean-data-dir=true

db-persistence.connection.db-name=uniconfig
db-persistence.connection.username=uniremote
db-persistence.connection.password=unipass
db-persistence.connection.uri=jdbc:postgresql://
db-persistence.connection.driver-class-name=org.postgresql.Driver
# This property controls the maximum number of milliseconds that a client will wait for a connection from the pool.
# If this time is exceeded without a connection becoming available, a {@link SQLException} will be thrown.
# Lowest acceptable connection timeout is 250 ms. Default: 30000 (30 seconds)
db-persistence.connection.connection-timeout=30000
# This property controls the maximum lifetime of a connection in the pool.
# On a connection-by-connection basis, minor negative attenuation is applied to avoid mass-extinction in the pool.
# We strongly recommend setting this value, and it should be several seconds shorter than any database or
# infrastructure imposed connection time limit. A value of 0 indicates no maximum lifetime (infinite lifetime),
# subject of course to the idleTimeout setting.
# The minimum allowed value is 30000ms (30 seconds). Default: 1800000 (30 minutes).
db-persistence.connection.max-lifetime=1800000
# This property controls the minimum number of idle connections that HikariCP tries to maintain in the pool.
# If the idle connections dip below this value and total connections in the pool are less than maximumPoolSize,
# HikariCP will make the best effort to add additional connections quickly and efficiently.
# However, for maximum performance and responsiveness to spike demands, we recommend not setting this value
# and instead allowing HikariCP to act as a fixed size connection pool. Default: same as maximumPoolSize
db-persistence.connection.min-idle-connections=10
# This property controls the maximum size that the pool is allowed to reach,
# including both idle and in-use connections. Basically this value will determine the maximum number
# of actual connections to the database backend. A reasonable value for this is best determined by your
# execution environment. When the pool reaches this size, and no idle connections are available,
# calls to getConnection() will block for up to connectionTimeout milliseconds before timing out.
db-persistence.connection.max-db-pool-size=20
# Timeout value used for socket read operations. If reading from the server takes longer than this value,
# the connection is closed. This can be used as both a brute force global query timeout and a method of
# detecting network problems. The timeout is specified in seconds and a value of 0 means that it is disabled.
db-persistence.connection.socket-read-timeout=20
# Enabled TLS authentication (if it is enabled, 'tlsClientCert', 'tlsClientKey', and 'tlsCaCert'
# are used and 'password' field is not used).
db-persistence.connection.enabled-tls=false
db-persistence.connection.tls-client-cert=./client.pks
db-persistence.connection.tls-client-key=./client.key
db-persistence.connection.tls-ca-cert=./ca.pks
# Password to an encrypted SSL certificate key file.
db-persistence.connection.ssl-password=

# List of network locations at which target database resides. The first entry is always tried in the first
# attempt during creation of database connection. If there are multiple entries specified, then other
# locations are used as fallback method in the order in which they are specified.

# database hostname / IP address
db-persistence.connection.database-locations[0].host=127.0.0.1
# TCP port on which target database listens to incoming connections
db-persistence.connection.database-locations[0].port=26257

# Repairs the Flyway schema history table before Flyway migration. This will perform the following actions:
# 1. Remove any failed migrations on databases.
# 2. Realign the checksums, descriptions and types of the applied migrations with available migrations.
db-persistence.connection.repair-schema-history=false
# Name / unique identifier of local Uniconfig instance.
#db-persistence.uniconfig-instance.instance-name=
# Hostname or IP address of a local host that runs Uniconfig.
db-persistence.uniconfig-instance.host=127.0.0.1
```

!!!
Be sure that [number of UniConfig nodes in cluster] * [maxDbPoolSize] does not exceed maximum allowed number
of open transactions and open connections on PostgreSQL side. Be aware that 'maxDbPoolSize' also caps maximum
number of open UniConfig transactions (1 UniConfig transaction == 1 database transaction == 1 connection to database).
!!!

#### UniConfig node identification

By default, UniConfig node name is generated randomly. This behaviour can be changed by setting
'db-persistence.uniconfig-instance.instance-name'. Instance name is leveraged, for example, in the clustering
of stream subscriptions.

Example:

```properties node identification properties
# Name / unique identifier of local Uniconfig instance.
# db-persistence.uniconfig-instance.instance-name=
# Hostname or IP address of a local host that runs Uniconfig.
db-persistence.uniconfig-instance.host=127.0.0.1
```

#### Kafka and notification properties

This section contains properties related to connections to Kafka brokers, Kafka publisher timeouts, authentication, 
subscription allocation, and rebalancing settings.

Example with essential properties:

```properties notification properties
# Grouped settings that are related to notifications.

# Flag that determines whether notifications are collected.
notifications.enabled=false
# How many unassigned subscriptions can be processed.
notifications.max-subscriptions-hard-limit=5000
# How many unassigned subscriptions can be processed within one subscription monitoring interval.
notifications.max-subscriptions-per-interval=10
# How often should UniConfig check for unassigned notifications subscriptions (in seconds).
notifications.subscriptions-monitoring-interval=5
# If response body should be included in notification.
notifications.audit-logs.include-response-body=false
# If calculating the diff result should be included in notification.
notifications.audit-logs.include-calculate-diff-result=false
# Maximum count of records, after reaching this count the oldest records will be deleted.
notifications.notification-db-threshold.max-count=10000
# Maximum age of records, all older records will be deleted (in hours).
notifications.notification-db-threshold.max-age=100

# Grouped settings that are related to kafka.

# Flag that determines whether audit logs are enabled.
notifications.kafka.audit-logs-enabled=true
# Flag that determines whether netconf notifications are enabled.
notifications.kafka.netconf-notifications-enabled=true
# Flag that determines whether gnmi notifications are enabled.
notifications.kafka.gnmi-notifications-enabled=true
# Flag that determines whether snmp notifications are enabled.
notifications.kafka.snmp-notifications-enabled=true
# Flag that sets the port to listen to notifications.
notifications.kafka.snmp-notifications-udp-port=50000
# The time in which the cache entry will expire (in seconds) if not accessed. This cache is used to map addresses
# from notifications to repository names that is used to parse the specific notification.
notifications.kafka.snmp-notifications-address-cache-duration=600
# The time in which the cache entry will expire (in seconds) if not accessed. This cache is used to map object
# identifiers to translated yang instance identifiers that are used to map them in the UniConfig schema context.
notifications.kafka.snmp-notifications-oid-cache-duration=600
# Flag that determines whether transaction notifications are enabled.
notifications.kafka.transaction-notifications-enabled=true
# Enabled collection and propagation of data-change-events into Kafka.
notifications.kafka.data-change-events-enabled=true
# Enabled collection and propagation of connection notifications into Kafka.
notifications.kafka.connection-notifications-enabled=true
# Unique identifier of topic that is used for storing netconf notifications.
notifications.kafka.netconf-notifications-topic-name=netconf-notifications
# Unique identifier of topic that is used for storing GNMI notifications.
notifications.kafka.gnmi-notifications-topic-name=gnmi-notifications
# Unique identifier of topic that is used for storing SNMP notifications.
notifications.kafka.snmp-notifications-topic-name=snmp-notifications
# Unique identifier of topic that is used for storing audit logs.
notifications.kafka.audit-logs-topic-name=audit-logs
# Unique identifier of topic that is used for storing transaction notifications.
notifications.kafka.transactions-topic-name=transactions
# Unique identifier of the Kafka topic used for distribution of data-change-events.
notifications.kafka.data-change-events-topic-name=data-change-events
# Unique identifier of the Kafka topic used for distribution of connection notifications.
notifications.kafka.connection-notifications-topic-name=connection-notifications
# If only connection notifications for NETCONF stream are enabled.
notifications.kafka.connection-notifications-netconf-stream-only=true
# The maximum thread pool size in the executor.
# A thread pool executor is needed to send messages to Kafka.
notifications.kafka.max-thread-pool-size=8
# The maximum capacity of the work queue in the executor.
notifications.kafka.queue-capacity=2048
# Unique identifier of the Kafka topic used for distribution of shell notifications.
notifications.kafka.shell-notifications-topic-name=shell-notifications
# Enabled shell notifications into Kafka.
notifications.kafka.shell-notifications-enabled=true
# List of Address / hostname of the interface on which Kafka broker is listening to incoming connections and
# TCP port on which Kafka broker is listening to incoming connections.
notifications.kafka.kafka-servers[0].broker-host=127.0.0.1
notifications.kafka.kafka-servers[0].broker-listening-port=9092
# Specifies the number of messages that the Kafka handler processes as a batch.
# In kafka is set with the parameter 'batch.size'.
notifications.kafka.kafka-producer.batch-size=16384
# Configuration of how long will the producer wait for the acknowledgement of a request. (in ms)
# If the acknowledgement is not received before the timeout elapses, the producer will resend the
# request or fail the request if retries are exhausted.
# In kafka is set with the parameter 'request.timeout.ms'.
notifications.kafka.kafka-producer.request-timeout=30000
# Configuration of the upper bound on the time to report success or failure after a
# call to send() returns.(in ms)
# This limits the total time that a record will be delayed prior to sending, the time to
# await acknowledgement from the broker (if expected), and the time allowed for retriable send failures.
# In kafka is set with the parameter 'delivery.timeout.ms'.
notifications.kafka.kafka-producer.delivery-timeout=120000
# Configuration of how long the send() method and the creation of connection for
# reading of metadata methods will block. (in ms).
# In kafka is set with the parameter 'max.block.ms'.
notifications.kafka.kafka-producer.blocking-timeout=60000
# The producer groups together any records that arrive in between request transmissions into a single batched
# request. Normally this occurs only under load when records arrive faster than they can be sent out.
# However, in some circumstances the client may want to reduce the number of requests even under moderate load.
# This setting accomplishes this by adding a small amount of artificial delay—that is, rather than immediately
# sending out a record, the producer will wait for up to the given delay to allow other records to be sent so
# that the sends can be batched together. This can be thought of as analogous to Nagle’s algorithm in TCP.
# This setting gives the upper bound on the delay for batching: once we get 'batch.size' worth of records for a
# partition it will be sent immediately regardless of this setting, however if we have fewer than this many
# bytes accumulated for this partition we will ‘linger’ for the specified time waiting for more records to show
# up. This setting defaults to 0 (i.e. no delay). Setting linger=5, for example, would have the effect of
# reducing the number of requests sent but would add up to 5ms of latency to records sent in the absence of
# load. In kafka is set with the parameter 'linger.ms'.
notifications.kafka.kafka-producer.linger=0
# Select compression algorithm for kafka producer.
# Can be one of: none gzip snappy lz4 zstd
# Snappy offers good balance and is good for JSON-based documents and logs.
notifications.kafka.kafka-producer.compression-type=SNAPPY
# The maximum time to block during receiving of next NETCONF notifications in milliseconds.
notifications.kafka.kafka-subscriber.netconf-notifications-poll-timeout=100
# The maximum time to block during receiving of next data-change-events in milliseconds.
notifications.kafka.kafka-subscriber.data-change-events-poll-timeout=100
# If this flag is set to 'true', then embedded Kafka is started during boot process.
# Otherwise, all other settings are effectively ignored.
notifications.kafka.embedded-kafka.enabled=false
# Directory to which embedded Kafka is downloaded and extracted.
notifications.kafka.embedded-kafka.install-dir=/tmp/embedded-kafka
# URL that is used for download of Kafka, if it hasn't been downloaded yet.
notifications.kafka.embedded-kafka.archive-url=https://dlcdn.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
# Directory to which embedded Kafka is downloaded and extracted.
notifications.kafka.embedded-kafka.data-dir=./data/embedded-kafka
# Clean data from previous run before starting of Kafka (= disabled persistence).
notifications.kafka.embedded-kafka.clean-data-before-start=true
# Number of partitions used for created topic.
notifications.kafka.embedded-kafka.partitions=1
```

### Load-balancer configuration

The following YAML code represents sample Traefik configuration that can be used in the clustered UniConfig deployment
(deployment with 1 Traefik node). There is one registered entry-point with identifier 'uniconfig' and port '8181'.

```yaml traefik configuration
api:
  insecure: true

entryPoints:
  uniconfig:
    address: ":8181"

providers:
  docker: {}
```

Next, it is needed to configure UniConfig docker containers with traefik labels - UniConfig nodes are automatically
detected by Traefik container as 'uniconfig' service providers. There is also URI prefix '/rests', name of the
'sticky cookie' 'uniconfig_server_id' and server port number '8181' (UniConfig web server is listening to incoming
HTTP requests on this port).

```yaml configuration of UniConfig docker container
services:
  uniconfig-1:
    container_name: "uc_1"
    labels:
      - traefik.enable=true
      - traefik.http.routers.uniconfig.entrypoints=uniconfig
      - traefik.http.routers.uniconfig.rule=PathPrefix(`/rests`)
      - traefik.http.services.uniconfig.loadBalancer.sticky.cookie.name=uniconfig_server_id
      - traefik.http.services.uniconfig.loadbalancer.server.port=8181
```

Values of all traefik labels should be same on all nodes in the cluster - scaling of UniConfig service in the cluster
(for example, using Docker Swarm tools) is simple since container settings do not change.

!!!
The similar configuration, like the presented one with Traefik, can be achieved using other load-balancer tools,
such as HAProxy.
!!!

## Clustering of NETCONF subscriptions and notifications

When device is installed with stream property set, subscriptions for all
provided streams are created in database. These subscriptions are always
created with UniConfig instance id set to null, so they can be acquired
by any UniConfig from cluster. Each UniConfig instance in cluster uses
its own monitoring system to acquire free subscriptions. Monitoring
system uses specialized transaction to lock subscriptions which prevents
more UniConfig instances to lock same subscriptions. While locking
subscription, UniConfig instance writes its id to subscription table to
currently locked subscription and which means that this subscription is
already acquired by this UniConfig instance. Other instances of
UniConfig will not find this subscription as free anymore.

### Optimal subscription count and rebalancing

With multiple UniConfig instances working in a cluster, each instance
calculates an optimal range of subscriptions to manage.

```
OPTIMAL_RANGE_LOW = (NUMBER_OF_ALL_SUBSCRIPTIONS / NUMBER_OF_LIVE_UC_NODES_IN_CLUSTER) * OPTIMAL_NETCONF_SUBSCRIPTIONS_APPROACHING_MARGIN
OPTIMAL_RANGE_HIGH = (NUMBER_OF_ALL_SUBSCRIPTIONS / NUMBER_OF_LIVE_UC_NODES_IN_CLUSTER) * OPTIMAL_NETCONF_SUBSCRIPTIONS_REACHED_MARGIN
# Example: optimal range for 5000 subscriptions with 3 nodes in the cluster and margins set to 0.05 and 0.10 = (1750, 1834)
```

Based on optimal range and a number of currently opened subscriptions,
each UniConfig node (while performing a monitoring system iteration)
decides whether it should:

- Acquire additional subscriptions before optimal range is reached
- Stay put and not acquire additional subscriptions in case optimal range is reached
- Release some of its subscriptions to trigger rebalancing until optimal range is reached

When an instance goes down, all of its subscriptions will be immediately
released and the optimal range for the other living nodes will changemanaged network devices
and thus the subscriptions will be reopened by the rest of the cluster.

!!!
There is a grace period before the other nodes take over the
subscriptions. So in case a node goes down and up quickly, it will
restart the subscriptions on its own.
!!!

Following example illustrates a timeline of a 3 node cluster and how
many subscriptions each node handles:

![notifications-in-cluster-rebalancing](netconf_subs_rebalancing.svg)

!!!
The hard limit still applies in clustered environment, and it will never
be crossed, regardless of the optimal range.
!!!
