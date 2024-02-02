# Logging framework

UniConfig uses the Logback logging framework. Logback is the successor to the log4j framework with many improvements, such as more options for configuration, better performance and context-based separation of logs. The latter is used widely in UniConfig to achieve per-device logging based on the set marker in the logs.

## Logback configuration

Logback configuration is placed in the **config/logback.xml** file under the UniConfig distribution. For more information, see [Logback configuration](http://logback.qos.ch/manual/configuration.html).

This section describes parts of the configuration in the context of UniConfig.

### Appenders

The following appenders are used:

1. `STDOUT` - Print logs to the console.
2. `logs` - Write all logs to the output file at **log/logs.log**. A rolling file appender is applied.
3. `netconf-notifications`, `netconf-messages`, `netconf-events`, `cli-messages` and `gnmi-messages` - Sifting appenders that split logs per node ID set in the marker of the logs. Logs are written to different subdirectories under the **log** directory, and are identified by their node ID. A rolling file appender is applied.
4. `restconf` - Appender used to write RESTCONF messages to the **log/restconf.log** file. A rolling file appender is applied.
5. `gnmi` - Appender used to write logs related to the gNMI topology. 

### Loggers

There are two groups of loggers:

1. Package-level logging brokers: Loggers used to write general messages to the console and a single output file.
    - Default logging level: `INFO`. For debugging purposes, it can be useful to change the logging level to `TRACE` or `DEBUG`.
    - Layers covered: UniConfig, Unified, Controller, RESTCONF, CLI, NETCONF, gNMI.
    - Appenders used: `STDOUT` and `logs`.
2. Loggers used for logging brokers: These loggers should not be changed since the state of logging can be changed using RPC calls. Classpaths point to specific classes that represent implementations of logging brokers.
    - Logging level is set to `TRACE`.
    - Appenders used: `netconf-notifications`, `netconf-messages`, `netconf-events`, `cli-messages`, `gnmi-messages` and `restconf`.

### Update configuration

Logback is configured to scan for changes in its configuration file and automatically reconfigure itself when the configuration file changes. The default scanning frequency is 5 seconds.

### Example configuration

In the **logback.xml** file, you can edit the level of logging for each UniConfig component:

```xml Logback
<configuration>
    <!-- root logger -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="logs"/>
    </root>

    <!-- UniConfig layer part -->
    <logger name="io.frinx.uniconfig" level="INFO"/>

    <!-- Unified layer part -->
    <logger name="io.frinx.unitopo" level="INFO"/>

    <!-- NETCONF part -->
    <logger name="org.opendaylight.netconf" level="INFO"/>

    <!-- Uniconfig property scanning -->
    <logger name="io.frinx.uniconfig.constants" level="INFO"/>

    <!-- CLI part -->
    <logger name="io.frinx.cli" level="INFO"/>

    <!-- SSH part (used by CLI and NETCONF) -->
    <logger name="org.apache.sshd" level="INFO"/>

    <!-- translation unit framework part -->
    <logger name="io.frinx.translate.unit.commons" level="INFO"/>
    <logger name="io.fd.honeycomb" level="INFO"/>

    <!-- gNMI part -->
    <logger name="io.frinx.gnmi" level="INFO" additivity="false">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="logs"/>
        <appender-ref ref="gnmi"/>
    </logger>

    <!-- RESTCONF part -->
    <logger name="org.opendaylight.restconf" level="INFO"/>
    <logger name="org.opendaylight.aaa" level="INFO"/>

    <!-- controller part -->
    <logger name="org.opendaylight.daexim" level="INFO"/>
    <logger name="org.opendaylight.controller" level="INFO"/>
    <logger name="org.opendaylight.yangtools" level="INFO"/>

    <!-- DOM mountpoint service -->
    <logger name="org.opendaylight.controller.md.sal.dom.broker.impl.mount" level="INFO"/>

    <!-- Kafka -->
    <logger name="org.apache.kafka.clients.NetworkClient" level="ERROR"/>

    <!-- Metrics -->
    <logger name="io.frinx.uniconfig.metrics" level="INFO" additivity="false">
        <appender-ref ref="metrics"/>
    </logger>

    <!-- CLI shell -->
    <logger name="io.frinx.uniconfig.shell" level="INFO"/>

    <!-- PostgreSQL driver -->
    <logger name="org.postgresql" level="INFO"/>
</configuration>
```

### Logging levels

The following logging levels are available:

#### INFO

This is the recommended logging level for production environments.

`INFO` messages describe the behavior of applications (for example, if a particular service starts or stops, or something is added to the database). During typical operations, these log entries are nothing to worry about and no follow-up actions are necessary.

#### DEBUG

The `DEBUG` level gives verbose diagnostic information that includes more detail than is necessary to use
the application. This logging level is used to diagnose, troubleshoot, or test an application to ensure that it runs smoothly.

#### TRACE

The `TRACE` level captures all detail about the application's behavior. It is mostly diagnostic and is more granular than `DEBUG`. This log level is used in situations where you need to see exactly what happened in your application.

## Logging brokers

The logging broker represents a configurable controller that logs one logical group of messages from a single classpath. Logging multiple messages from the same classpath simplifies the configuration of loggers in
Logback, since only one logger per broker must be specified.

The logging broker is controlled using RESTCONF RPCs. There are several operations that can trigger logging for the whole broker or only for specified node IDs. The logger configuration in the logback file assigned to the broker should not be changed.

The following subsections describe available logging brokers:

#### RESTCONF

- Used to log authenticated HTTP requests and responses; information about URI, source, HTTP method, query parameters, HTTP headers and body.
- Per-device logging cannot be enabled for this broker. All logs are saved to the **log/restconf.log** file.
- It is possible to specify HTTP headers whose content is masked the in logs with asterisks (`*`). This is useful if some headers contain private data (such as `Authorization` or a `Cookie` headers). Hidden HTTP headers are marked using header identifiers.
- It is also possible to configure HTTP methods for which communication (requests and responses) is not logged to a file.
- Requests and responses are paired using a unique `message-id` attribute, which is not part of the HTTP request but is generated on the RESTCONF server.
- Requests and responses contain Uniconfig transactions for easier matching with log transactions.

**Example** - Request and corresponding response (same `message-id`):

```
08:51:21.508 TRACE org.opendaylight.restconf.nb.rfc8040.jersey.providers.logging.RestconfLoggingBroker - HTTP request:
Message ID: 3
Uniconfig transaction: b6639cb4-55f2-449e-a91e-d2ad490198d2
HTTP method: POST
URI: http://localhost:8181/rests/operations/logging:enable-device-logging
Source address: 0:0:0:0:0:0:0:1
Source port: 37472
User ID: admin@sdn
HTTP headers:
    User-Agent: [curl/7.69.1]
    Authorization: ***
    Host: [localhost:8181]
    Accept: [*/*]
    Content-Length: [116]
    Content-Type: [application/json]
Request body:
{
  "input": {
    "broker-identifier": "netconf_notifications",
    "device-list": [
      "xr6",
      "xr7"
    ]
  }
}

08:51:21.518 TRACE org.opendaylight.restconf.nb.rfc8040.jersey.providers.logging.RestconfLoggingBroker - HTTP response:
Request message ID: 3
Uniconfig transaction: b6639cb4-55f2-449e-a91e-d2ad490198d2
Status code: 204
HTTP headers:
    Content-Type: [application/yang-data+json]
Response body:
```

#### CLI messages

- Used to log all CLI requests and responses.
- CLI requests and responses are paired with a unique `message-id` attribute.
- Supports per-device logging. Logs for CLI messages are stored under the **log/cli-messages** directory and named according to the pattern `[node-id].log`.

**Example** - Send POST RPC to install a CLI device and pair requests with corresponding responses (same `message-id`):

```
15:11:33.119 TRACE io.frinx.cli.io.impl.cli.CliLoggingBroker - Message-ID:1 - Sending CLI command:
show configuration commit list | utility egrep "^1 "
15:11:33.697 TRACE io.frinx.cli.io.impl.cli.CliLoggingBroker - Message-ID:1 - Received CLI response:
Wed Sep 22 13:11:29.776 UTC
1    1000005360            cisco     vty0:node0_0_CPU0   CLI         Mon Sep  6 07:53:03 2021
15:11:33.724 TRACE io.frinx.cli.io.impl.cli.CliLoggingBroker - Message-ID:2 - Sending CLI command:
show running-config
15:11:34.459 TRACE io.frinx.cli.io.impl.cli.CliLoggingBroker - Message-ID:2 - Received CLI response:
Wed Sep 22 13:11:30.116 UTC
Building configuration...
!! IOS XR Configuration 5.3.4
!! Last configuration change at Mon Sep  6 07:53:03 2021 by cisco
!
hostname XR5
interface MgmtEth0/0/CPU0/0
 ipv4 address 192.168.1.214 255.255.255.0
!
interface GigabitEthernet0/0/0/0
 shutdown
!
interface GigabitEthernet0/0/0/1
 description init description
 shutdown
!
interface GigabitEthernet0/0/0/2
!
ssh server v2
ssh server netconf port 830
netconf-yang agent
 ssh
!
end
```

#### NETCONF Messages

- Used to log all NETCONF messages, incoming and outgoing, except for NETCONF notifications (a separate broker handles notifications).
- NETCONF RPCs and responses can be matched using the `message-id` attribute placed in the RPC header.
- Supports per-device logging. Logs for NETCONF messages are stored in the **log/netconf-messages** directory and named according to the pattern `[node-id].log`.

**Example** - Send NETCONF GET RPC and receive a response:

```
11:10:15.038 TRACE org.opendaylight.netconf.logging.brokers.NetconfMessagesLoggingBroker - Session: 641 - Sending NETCONF message:
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-179">
    <get>
        <filter xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0" ns0:type="subtree">
            <netconf xmlns="urn:ietf:params:xml:ns:netmod:notification"/>
        </filter>
    </get>
</rpc>

11:10:15.055 TRACE org.opendaylight.netconf.logging.brokers.NetconfMessagesLoggingBroker - Session: 641 - Received NETCONF message:
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-179">
    <data>
        <netconf xmlns="urn:ietf:params:xml:ns:netmod:notification">
            <streams>
                <stream>
                    <name>NETCONF</name>
                    <description>default NETCONF event stream</description>
                    <replaySupport>true</replaySupport>
                    <replayLogCreationTime>2020-09-29T09:49:54+00:00</replayLogCreationTime>
                </stream>
                <stream>
                    <name>oam</name>
                    <description>Vendor notifications</description>
                    <replaySupport>true</replaySupport>
                    <replayLogCreationTime>2020-11-09T13:20:01+00:00</replayLogCreationTime>
                </stream>
            </streams>
        </netconf>
    </data>
</rpc-reply>
```

!!!
The number `641` represents the session ID read from the NETCONF hello message. If multiple sessions are created between the NETCONF server and client that are logically grouped under the same node ID, logs from multiple sessions are stored in the same logging file. This is necessary to distinguish between the sessions. Multiple NETCONF sessions between the UniConfig and NETCONF server are created for each subscription to the NETCONF stream.
!!!

#### NETCONF Notifications

- Used to log incoming NETCONF notifications.
- Supports per-device logging. Logs for NETCONF notifications are stored in the **log/netconf-notifications** directory and named according to the pattern `[node-id].log`.

**Example** - Receiving two notifications:

```
11:37:03.907 TRACE org.opendaylight.netconf.logging.brokers.NotificationsLoggingBroker - 117123a1: Received NETCONF notification:
<notification xmlns="urn:ietf:params:xml:ns:netconf:notification:1.0">
    <eventTime>2020-11-11T02:36:54.484233-08:00</eventTime>
    <netconf-session-start xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-notifications">
        <username>admin</username>
        <session-id>35</session-id>
        <source-host>10.255.246.31</source-host>
    </netconf-session-start>
</notification>

11:37:04.515 TRACE org.opendaylight.netconf.logging.brokers.NotificationsLoggingBroker - 117123a1: Received NETCONF notification:
<notification xmlns="urn:ietf:params:xml:ns:netconf:notification:1.0">
    <eventTime>2020-11-11T02:36:55.098119-08:00</eventTime>
    <netconf-session-end xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-notifications">
        <username>admin</username>
        <session-id>35</session-id>
        <source-host>10.255.246.31</source-host>
        <termination-reason>dropped</termination-reason>
    </netconf-session-end>
</notification>
```

#### NETCONF Events

- Used to log session-related information about establishing or closing a NETCONF session from the view of the NETCONF client placed in UniConfig.
- These logs do not contain full printouts of sent or received NETCONF messages.
- Supports per-device logging. Logs for NETCONF events are stored in the **log/netconf-events** directory and according to the pattern `[node-id].log`.

**Example**:

```
11:08:59.407 INFO org.opendaylight.netconf.logging.brokers.NetconfEventsLoggingBroker - node1: Connecting remote device with config: Node{getNodeId=Uri [_value=node1], augmentations={interface org.opendaylight.yang.gen.v1.urn.opendaylight.netconf.node.topology.rev150114.NetconfNode=NetconfNode{getActorResponseWaitTime=5, getBetweenAttemptsTimeoutMillis=2000, getConcurrentRpcLimit=0, getConnectionTimeoutMillis=60000, getCredentials=LoginPassword{getPassword=versa123, getUsername=admin, augmentations={}}, getCustomizationFactory=default, getDefaultRequestTimeoutMillis=60000, getDryRunJournalSize=0, getEditConfigTestOption=Set, getHost=Host [_ipAddress=IpAddress [_ipv4Address=Ipv4Address [_value=10.103.5.202]]], getKeepaliveDelay=5, getMaxConnectionAttempts=100, getPort=PortNumber [_value=2022], getSleepFactor=1.0, getYangModuleCapabilities=YangModuleCapabilities{getCapability=[http://tail-f.com/ns/docgen/experimental1?module=docgen&amp;revision=2019-11-01], isOverride=false, augmentations={}}, isEnabledNotifications=true, isReconnectOnChangedSchema=false, isSchemaless=false, isTcpOnly=false}, interface org.opendaylight.yang.gen.v1.http.frinx.io.yang.uniconfig.config.rev180703.UniconfigConfigNode=UniconfigConfigNode{getBlacklist=Blacklist{getExtension=[tailf:display-when false], augmentations={}}, isUniconfigNativeEnabled=true}}}
11:09:00.554 DEBUG org.opendaylight.netconf.logging.brokers.NetconfEventsLoggingBroker - node1: Session established
11:09:00.558 DEBUG org.opendaylight.netconf.logging.brokers.NetconfEventsLoggingBroker - node1: Session advertised capabilities: ...
11:09:00.832 DEBUG org.opendaylight.netconf.logging.brokers.NetconfEventsLoggingBroker - node1: Connector for node created successfully
```

#### gNMI Messages

- Used to log all gNMI SET/GET messages, incoming or outgoing, except for gNMI notifications.
- Support per-device logging. Logs for gNMI messages are stored in the **log/gnmi-messages** directory and named according to the pattern `[node-id].log`.

**Example** - Send gNMI SET request and receive a response:

```
12:12:28.658 TRACE io.uniconfig.gnmi.logging.brokers.GnmiMessagesLoggingBroker - Message-id: 1693479982709, sending gNMI SET message:
replace {
  path {
    elem {
      name: "openconfig-network-instance:network-instances"
    }
    elem {
      name: "network-instance"
      key {
        key: "name"
        value: "default"
      }
    }
    elem {
      name: "protocols"
    }
    elem {
      name: "protocol"
      key {
        key: "identifier"
        value: "BGP"
      }
      key {
        key: "name"
        value: "bgp"
      }
    }
    elem {
      name: "bgp"
    }
    elem {
      name: "global"
    }
    elem {
      name: "logging-options"
    }
    elem {
      name: "config"
    }
  }
  val {
    json_ietf_val: "{\"config\":{\"log-neighbor-state-changes\":true}}"
  }
}

12:12:28.878 TRACE io.uniconfig.gnmi.logging.brokers.GnmiMessagesLoggingBroker - Message-id: 1693479982709, received gNMI SET message:
response {
  path {
    elem {
      name: "openconfig-network-instance:network-instances"
    }
    elem {
      name: "network-instance"
      key {
        key: "name"
        value: "default"
      }
    }
    elem {
      name: "protocols"
    }
    elem {
      name: "protocol"
      key {
        key: "identifier"
        value: "BGP"
      }
      key {
        key: "name"
        value: "bgp"
      }
    }
    elem {
      name: "bgp"
    }
    elem {
      name: "global"
    }
    elem {
      name: "logging-options"
    }
    elem {
      name: "config"
    }
  }
  op: REPLACE
}
timestamp: 1692266952617002464
```

### Supported logging settings

Current logging broker settings are stored in the `Operational` datastore under the `logging-status` root container.

The following example shows a GET query that displays the logging broker settings:

```bash GET Logging Status
curl --location --request GET 'http://127.0.0.1:8181/rests/data/logging-status?content=nonconfig' \
--header 'Accept: application/json'
```

**Response:**

```json Status:
{
  "logging-status": {
    "broker": [
      {
        "broker-identifier": "netconf_messages",
        "is-logging-broker-enabled": true,
        "is-logging-enabled-on-all-devices": true,
        "enabled-devices": [
          "xr6",
          "xr7"
        ]
      },
      {
        "broker-identifier": "restconf",
        "is-logging-broker-enabled": true,
        "restconf-logging:hidden-http-methods": [
          "GET"
        ],
        "restconf-logging:hidden-http-headers": [
          "Authorization",
          "Cookie"
        ]
      },
      {
        "broker-identifier": "netconf_notifications",
        "is-logging-broker-enabled": true,
        "is-logging-enabled-on-all-devices": true
      },
      {
        "broker-identifier": "netconf_events",
        "is-logging-broker-enabled": true,
        "is-logging-enabled-on-all-devices": true
      },
      {
        "broker-identifier": "gnmi_messages",
        "logging-enabled-on-all-devices": true,
        "logging-broker-enabled": true,
        "gnmi-logging:message-types": [
          "SET"
        ]
      },
      {
        "broker-identifier": "shell_logs",
        "logging-enabled-on-all-devices": true,
        "logging-broker-enabled": true
      },
      {
        "broker-identifier": "cli_messages",
        "logging-enabled-on-all-devices": true,
        "logging-broker-enabled": true
      }
    ],
    "global": {
      "hidden-types": [
        "password",
        "encrypted"
      ]
    }
  }
}
```

Logging settings are encapsulated inside multiple list entries (`broker` list). Each entry contains settings for one logging broker.

Settings placed under a single logging entry:

- `broker-identifier` - Unique identifier for the logging broker. The following brokers are supported: `netconf\_messages`, `restconf`, `netconf\_notifications`, `netconf\_events`, and `cli\_messages`.
- `is-logging-broker-enabled` - Specifies if the logging broker is enabled or not. If the broker is disabled, no logging messages are generated.
- `is-logging-enabled-on-all-devices` - If set to `true`, logs are separated into distinct files in the scope of all devices. If set to `false`, logging is enabled only for devices listed in the `enabled-devices` leaf-list/array. This setting is not supported for the `restconf` logging broker as RESTCONF currently does not differentiate the node ID in requests or responses.
- `enabled-devices` - If `is-logging-enabled-on-all-devices` is set to `false`, logs are generated only for devices specified in this list. Acts as a simple filtering mechanism based on the whitelist. A blacklist approach is not supported, as it is not possible to set `is-logging-enabled-on-all-devices` to `true` and specify devices for which logging feature is disabled. This setting is not supported for the `restconf` logging broker.

RESTCONF-specific settings:

- `restconf-logging:hidden-http-methods` - HTTP requests and associated HTTP responses are not logged if the  request's HTTP method is included in this list. Names of HTTP methods must be specified in uppercase.
- `restconf-logging:hidden-http-headers`` - List of HTTP headers (names of headers) whose content is hidden in the logs. Names of headers are case-insensitive.

GNMI-specific settings:

- `gnmi-logging:message-types` - gNMI message types that are logged. Names of message types must be specified in uppercase.

Global settings common for all logging brokers:

- `hidden-types` - The value of leafs or leaf-lists using one of the types specified is masked in the logs with asterisks (`*`). Useful for hiding passwords and other confidential data in logs.

### Initial configuration

By default, all logging brokers are disabled and logging is disabled on all devices. To enable per-device logging, you must explicitly specify a list of devices. Additionally, RESTCONF-specific filtering is not configured, all HTTP requests and responses are fully logged and no content is dismissed. By default, only the `SET` gNMI message type is set to be logged.

The initial logging configuration can be adjusted by adding the `logging-controller` configuration to the
**config/application.properties** file. The structure of this configuration section conforms to the YANG structure described by the `logging` and `restconf-logging` modules. It is possible to copy the state of the `Operational` datastore under `logging-status` into the `logging-controller` node.

The following properties snippet shows the sample configuration
`logging-controller`. The logging brokers `netconf\_messages` and
`netconf\_notifications` are enabled, (`netconf\_messages` for all devices and
`netconf\_notifications` only for `xr6` and `xr7` devices).

!!!
If unknown parameters are specified in a configuration file, they are ignored and a warning is logged.
!!!

```properties
# Logging controller config
#   Logging all NETCONF messages.
logging-controller.brokers[0].broker-identifier=netconf_messages
logging-controller.brokers[0].logging-broker-enabled=true
logging-controller.brokers[0].logging-enabled-on-all-devices=true

#   Logging all NETCONF notification messages.
logging-controller.brokers[1].broker-identifier=netconf_notifications
logging-controller.brokers[1].logging-broker-enabled=true
logging-controller.brokers[1].logging-enabled-on-all-devices=false
logging-controller.brokers[1].device-list[0]=xr6
logging-controller.brokers[1].device-list[1]=xr7
```

### Control logging using RPC calls

As logging settings are stored in the `Operational` datastore, it is possible to adjust these settings in runtime only using RPC calls. The following subsections describe available RPCs.

#### Enable logging broker

This RPC is used to enable the logging broker, which is then available to write logs. RPC input contains only the name of the logging broker (`broker-identifier`).

**Example** - Enable logging broker with the `restconf` identifier:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:enable-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "restconf"
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Disable logging broker

This RPC is used to disable a logging broker, which will no longer write any logs regardless of other settings. RPC input contains only the name of the logging broker.

**Example** - Disable logging broker with the `restconf` identifier:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:disable-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "restconf"
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Enable default device logging

This RPC is used to set the default device logging to `true`. Logs are written for all devices without filtering based on node ID.

RPC input contains only the name of the logging broker (`broker-identifier`). Invoking this RPC clears the leaf-list `enabled-devices`.

**Example** - Enable default device logging in the `netconf\_messages` logging broker:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:enable-default-device-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "netconf_messages"
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Disable default device logging

This RPC is used to set the default device logging to `false`. Logs are written only for devices named in the `enabled-devices` leaf-list. If `enabled-devices` does not contain a node ID, logging in the corresponding broker is effectively turned off.

RPC input contains only the name of the logging broker (`broker-identifier`).

**Example** - Disable default device logging in the `netconf\_messages` logging broker:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:disable-default-device-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "netconf_messages"
    }
}'
```

The output shows a positive response:

```json RPC Response, Status: 204
```

#### Enable device logging

This RPC is used to enable logging for specified devices identified by node IDs. RPC input contains the name of the logging broker (`broker-identifier`) and a list of node IDs (`device-list`).

**Example** - Enable logging for devices with node IDs `node1`, `node2`, and `node3` in the `netconf\_events` logging broker:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:enable-device-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "netconf_events",
        "device-list": [
            "node1",
            "node2",
            "node2"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Disable device logging

This RPC is used to turn off logging for specified devices identified by node IDs. RPC input contains the name of the logging broker (`broker-identifier`) and a list of node IDs (`device-list`).

**Example** - Disable logging for device with node ID `node1` in the `netconf\_events` logging broker:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:disable-device-logging' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "broker-identifier": "netconf_events",
        "device-list": [
            "node1"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Set global hidden types

This RPC is used to set identifiers of hidden YANG type definitions. Values of leaves and leaf-lists described by these types are masked in output logs. Overwrites all previously configured hidden types. An empty list of hidden types will disable filtering of data values. Filtering applies to all logs, including RESTCONF logs.

**Example** - Set three (3) hidden types:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/logging:set-global-hidden-types' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "hidden-types": [
            "types:password",
            "types:encrypted",
            "types:hash"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Set Hidden HTTP Headers

This RPC is used to overwrite the list of HTTP headers whose content is hidden in the output of RESTCONF logs. Only affects the `restconf` logging broker.

HTTP headers in both requests and responses are masked. The list of hidden headers denotes header identifiers. The identifier of hidden headers is still included in output logs, but their content is masked with asterisks (`*`).

**Example** - Hide the content of `Authorization` and `Cookie` HTTP headers:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/restconf-logging:set-hidden-http-headers' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "hidden-http-headers": [
            "Authorization",
            "Cookie"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Set hidden HTTP methods

This RPC is used to overwrite the list of HTTP methods. RESTCONF communication, which may include invoking hidden HTTP methods, is not displayed in output logs.

Both requests and responses with hidden HTTP methods are excluded from the log files. Only affects the `restconf` logging broker.

**Example** - Hide `GET` and `PATCH` communication in RESTCONF logs:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/restconf-logging:set-hidden-http-methods' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "hidden-http-methods": [
            "GET",
            "PATCH"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```

#### Set gNMI message types

This RPC is used to overwrite the list of supported gNMI message types. Only affects the logging of gNMI messages.

**Example** - Set 'SET' and 'GET' message types:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/gnmi-logging:set-message-types' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "message-types": [
            "SET", "GET"
        ]
    }
}'
```

The output shows a successful response:

```json RPC Response, Status: 204
```
