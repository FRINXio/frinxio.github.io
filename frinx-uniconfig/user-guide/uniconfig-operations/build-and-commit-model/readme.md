# Build-and-Commit Model

The Build-and-Commit model is based on explicit creation of a transaction,
invoking operations in the scope of this transaction and, finally,
committing or closing the transaction. The transaction represents a session
between client and UniConfig instance.

Using explicitly created transactions has multiple advantages in
comparison to the Immediate Commit Model:

- Multiple operations and modifications can be invoked in a single
    transaction while keeping transactions isolated.
- Most UniConfig operations, such as calculate-diff and commit, have no use in
    the Immediate Commit Model. They are valuable only if the Build-and-Commit
    Model is used.
- The transaction allows a client to identify if it still communicates with the
    same UniConfig instance (this property is usable in the clustered
    deployment). If the UniConfig instance does not know about the transaction,
    the request will fail because the transaction expired, is closed or was
    never created in the first place.

## Configuration

Configurations related to UniConfig transactions are placed in the
**config/lighty-uniconfig-config.json** file under the `transactions` container.
Note that the Build-and-Commit model is enabled if `uniconfigTransactionEnabled`
is set to `true` (this is the default value).

```
// Grouped settings that are related to Uniconfig transactions.
"transactions": {
    /*
    Time after transaction can be closed [seconds] by transaction cleaner.
    */
    "transactionIdleTimeOut": 300,
    /*
    Maximum transaction age before it can be evicted from transaction registry [seconds].
    Configuring '0' disables cleaning of Uniconfig transactions.
    */
    "maxTransactionAge": 1800,
    /*
    Interval at which expired transactions are closed and cleaned [seconds].
    Expired transaction: transaction which age exceeds 'maxTransactionAge' setting.
    Only dedicated Uniconfig transactions (initialized using 'create-transaction' RPC)
    are cleaned - shared transaction is never removed or invalidated.
     Configuring '0' disables cleaning of Uniconfig transactions.
    */
    "cleaningInterval": 60,
    /*
    Boolean value if the Immediate Commit Model is enabled or not. Default value is true.
    If disabled, only manually created transactions can exist.
    */
    "isImmediateCommitEnabled": true
}
```

## Optimistic locking mechanism

Race conditions between transactions that are committed in parallel and contain
changes to the same nodes (uniconfig, unistore, snapshot or template nodes) are
solved using the optimistic locking mechanism. The configuration of a node can
be modified in parallel from two transactions, but only the first committed
transaction will succeed. The commit for the second transaction will fail.

UniConfig uses two different techniques to detect conflicts during commit or
checked-commit operations:

1. **Comparing configuration fingerprints** - The fingerprint value for an
altered node is updated at the end of the commit operation. At the beginning of
a commit operation, UniConfig compares the value of the actual fingerprint in
the database with the value of the fingerprint read before the first CRUD
operation done in the transaction and the last synced fingerprint (updated after
execution of the **sync-from-network RPC**). If the actual fingerprint in the
database is equal to the fingerprint read before the first CRUD operation or the
last synced fingerprint, the commit operation can continue. Otherwise, an error
is returned without touching any devices in the network.

2. **Per-node advisory locks** - Comparison of configuration fingerprints is
reliable if transactions are committed one after another. However, such
serialization cannot be achieved in the clustered environment as UniConfig
instances are not coordinated. If two transactions are committed at the same
time and both assume that configuration fingerprints haven't been updated by
another transaction, both transactions may start to push changes to network
devices simultaneously. To prevent this scenario, UniConfig locks the node in
the PostgresSQL database using transaction-level advisory locks at the beginning
of the commit operation. If another transaction tries to lock the same node, its
attempt will fail, and the second transaction cannot enter the critical section
but will fail. Locks are automatically released at the end of the transaction
(**commit RPC** closes the transaction).

All possible scenarios are captured in the following diagrams.

![Optimistic locking](optimistic-locking.svg)

## Dynamic mountpoints

Mountpoints are created only when UniConfig needs to read / write some
data from / to device and lifecycle of mountpoint is bounded by
lifecycles of transactions that use the mountpoint. If some mountpoint
is not used by any transaction, then UniConfig automatically closes this
mountpoint - associated operational data on southbound layer and
connection to device are removed.

The first diagram demonstrates mounting of 2 devices which are used by 1
transaction - after this transaction is closed, both mountpoints are
closed. The second diagram shows scenario in which 2 transactions share
1 of 2 mountpoints - after the first transaction is closed, 1 of the
mountpoints is not closed since the second transaction still may
communicate with corresponding device.

![Dynamic mountpoints](dynamic-mountpoints.svg)

## Creation of transaction

Transaction can be created using create-transaction RPC. RPC doesn't
specify input body and also returns response without body. Response
additionally contains Set-Cookie header with UNICONFIGTXID key and
corresponding value - transaction identifier that conforms RFC-4122
Universally Unique IDentifier (UUID) format.

Process of transaction creation is depicted by following sequence
diagram.

![create-transaction RPC](create-transaction-rpc-Creation_of_transaction__RPC_.svg)

UniConfig is performing following steps after calling create-transaction
RPC:

1. Creation of connection to database system - Connection is created
    with disabled auto-commit - enabling transactional features.
    UniConfig uses 'read committed' isolation level.
2. Creation of database transaction - It provides access to remote
    PostgreSQL database. Using database transaction it is possible to
    read committed data, read uncommitted changes created by this
    transaction and write modifications to database. Data read at the
    first access to some resource is cached to datastore transaction -
    when some component tries to access the same resource again, it is
    read only from datastore transaction. Data is written to database
    transaction at invocation of commit/checked-commit RPC.
3. Creation of datastore read-write transaction - It provides access to
    OPER and CONFIG datastores bound to this transaction. Datastore is
    used only as a cache between application and PostgreSQL database,
    and it resides only in the memory allocated to UniConfig process.
    Datastore transaction is never committed - cache is trashed at the
    end of the transaction life.
4. Registration of transaction - Transaction is always bound to 1
    specific UniConfig instance.

### Successful example

The following request shows successful creation of UniConfig
transaction. Response contains Set-Cookie header with UNICONFIGTXID key
and value.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Accept: application/json'
```

```text RPC Response, Status: 201
1Set-Cookie: UNICONFIGTXID=2ab7cfc3-dedc-4444-8431-6e9cf94fad3b;Version=1;Comment="uniconfig transaction created";Path=/rests/
```

### Failed example

The most common reason for failed creation of UniConfig transaction is
reached maximum number of open transactions that is limited by
('maxDbPoolSize' - 'maxInternalDbConnections') database connection pool setting. In that case,
UniConfig returns response with 500 status code.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-type": "protocol",
        "error-message": "Failed to create Uniconfig transaction",
        "error-tag": "operation-failed",
        "error-info": "Maximum open transactions created by User was reached"
      }
    ]
  }
}
```

### Transaction idle-timeout

Create-transaction RPC can be used with optional query parameter called timeout.
This parameter is used to override global idle timeout for transaction created
by this RPC call. After transaction inactivity for specified time transaction will be
automatically cleaned. Value of this parameter is whole number and defines
time in seconds.

```URL Example request with timeout parameter
http://localhost:8181/rests/operations/uniconfig-manager:create-transaction?timeout=100
```

### Dedicated session to device

By default, UniConfig shares a southbound session to a network device if
multiple UniConfig transactions use the same device via the same management
protocol. This behaviour can be disabled using the `dedicatedDeviceSession`
query parameter, which accepts a boolean value. Afterwards the UniConfig
transaction will create a dedicated session to the device, which is used only by
one transaction and is closed immediately after committing or closing the
transaction.

Dedicated sessions to a device are useful when:
* The evice is not able to process requests in parallel via the same session.
* The device is able to process requests in parallel via same session, but does
  not process them in parallel, decreasing processing performance.

```URL Setting dedicated device sessions
http://127.0.0.1:8181/rests/operations/uniconfig-manager:create-transaction?dedicatedDeviceSession=true
```
### Audit parameters

Uniconfig can pass audit parameters to network devices when a Uniconfig transaction is committed.

Following implementations are currently supported on southbound:

* **Tailf confD extension (NETCONF)**
```
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-3">
    <commit>
        <confirmed/>
        <rollback-comment xmlns="http://tail-f.com/ns/netconf/rollback">Uniconfig transaction: c7be0e5d-2bc3-473f-8590-0fe3f732beef</rollback-comment>
        <rollback-label xmlns="http://tail-f.com/ns/netconf/rollback">test</rollback-label>
    </commit>
</rpc>
```

On the northbound, the parameters are exposed as HTTP headers:

* AUDIT_USER
* AUDIT_COMMENT

```
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'AUDIT_USER: audit user' \
--header 'AUDIT_COMMENT: audit comment'
```

## Invocation of CRUD operation in transaction

CRUD operations for modification or reading node configuration can be
invoked in the specific transaction by appending UNICONFIGTXID (key)
with UUID of transaction (value) to Cookie headers. In that case,
operation will be invoked only in the scope of single transaction -
changes are not visible to other transactions until this transaction is
successfully committed.

Next diagram describes execution of CRUD operation from RESTCONF API. It
shows also difference between datastore and database transaction - data
is read from database only at the first access to some data (for
example, node configuration). After that, this configuration is cached
inside temporary datastore transaction - goal is to improve performance
by limiting transferring data between UniConfig and PostgreSQL. Next
access to same configuration can be evaluated under in-memory datastore.

![Invocation of CRUD](crud-operation-in-transaction-Invocation_of_CRUD_operation_in_UniConfig_transaction.svg)

### Successful example

The following request demonstrates reading of some configuration from
uniconfig topology, junos node in the transaction with ID
'd7ff736e-8efa-4cc5-9d27-b7f560a76ff3'.

```bash GET Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=junos/configuration/frinx-openconfig-interfaces:interfaces/interface=fxp0' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=d7ff736e-8efa-4cc5-9d27-b7f560a76ff3'
```

```json GET Response, Status: 200
{
    "frinx-openconfig-interfaces:interface": [
        {
            "name": "fxp0",
            "config": {
                "type": "iana-if-type:ethernetCsmacd",
                "enabled": true,
                "name": "fxp0"
            },
            "subinterfaces": {
                "subinterface": [
                    {
                        "index": 0,
                        "config": {
                            "index": 0
                        },
                        "frinx-openconfig-if-ip:ipv4": {
                            "addresses": {
                                "address": [
                                    {
                                        "ip": "10.103.5.208",
                                        "config": {
                                            "prefix-length": 24,
                                            "ip": "10.103.5.208"
                                        }
                                    }
                                ]
                            }
                        }
                    }
                ]
            }
        }
    ]
}
```

### Failed example

Trying to use non-existing UniConfig transaction results in 422 status
code (Unprocessable Entity).

```bash GET Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=junos/configuration/frinx-openconfig-interfaces:interfaces/interface=fxp0' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=d7ff736e-8efa-4cc5-9d27-b7f560a76ff4'
```

```json GET Response, Status: 422
{
  "errors": {
    "error": [
      {
        "error-message": "Unknown uniconfig transaction: d7ff736e-8efa-4cc5-9d27-b7f560a76ff4",
        "error-tag": "invalid-transaction",
        "error-type": "protocol"
      }
    ]
  }
}
```

## Invocation of RPC operation in transaction

RPC operation can be invoked in the specific transaction the same way as
CRUD operation - by specification of UNICONFIGTXID in the Cookie header.

There are few differences between CRUD and RPC operations from the view
of transactions:

- Commit, checked-commit, and close-transaction RPCs can state of the
    transaction. Create-transaction RPC is reserved for creation of
    transaction.
- Not all RPC operations that are exposed by UniConfig use dedicated
    transactions - in that case, these RPCs just ignore explicitly
    specified transaction and either don't work with transactions at all
    or create transaction internally (examples: install-node,
    uninstall-node RPC).
- There are also transaction-aware operations that directly leverage
    properties of transactions. For example, if some UniConfig RPC is
    invoked with empty list of target nodes, then operation is
    automatically applied to all modified nodes in the transaction
    (calculate-diff RPC with empty target nodes computes diff for all
    modified nodes in the transaction).

Following diagram shows execution of random RPC in the specified
transaction.

![Invocation of RPC](invoke-operation-in-transaction-Invocation_of_UniConfig_RPC_operation_in_transaction.svg)

### Successful example

Invocation of calculate-diff RPC in the transaction which contains
modifications done on the 'junos' node.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=d7ff736e-8efa-4cc5-9d27-b7f560a76ff3' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["junos"]
        }
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "junos",
          "topology-id": "uniconfig",
          "updated-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=junos/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=fxp0/config",
              "data-actual": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": true,\n    \"name\": \"fxp0\"\n  }\n}",
              "data-intended": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"fxp0\"\n  }\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Failed example

Invocation of calculate-diff RPC with transaction ID that has wrong
format.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=d7ff736e-8efa-4cc5-9d27-b7f560a76ffx' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["junos"]
        }
    }
}'
```

```json RPC Response, Status: 403
{
  "errors": {
    "error": [
      {
        "error-type": "protocol",
        "error-message": "Invalid transaction-id format; it should conform UUID based on RFC 4122: 2a335e04-ae11-4677-885b-cea1ea71157x",
        "error-tag": "access-denied",
        "error-info": "Error at index 11 in: \"cea1ea71157x\""
      }
    ]
  }
}
```

## Closing transaction

There are 2 options how transaction can be closed:

1. close-transaction RPC - Explicit closing of transaction that results
    in dropping of all changes done in the transaction.
2. commit/checked-commit RPC - After execution of commit operation,
    transaction is automatically closed (despite of commit result).
    Behaviour of commit and checked commit RPC is described in better
    detail under the 'UniConfig Node Manager' section.

Close-transaction RPC doesn't contain body, only Cookie header with
UNICONFIGTXID property pointing to transaction that user would like to
close. Response contains information if transaction has been
successfully closed.

Following sequence diagrams describe close-transaction procedure. It is
split into 2 diagrams to improve readability and to reuse some parts
from other diagrams.

![close-transaction RPC](close-transaction-rpc-Closing_transaction__RPC_.svg)

![Clean orphaned mountpoints](clean-orphaned-mountpoints-Clean_orphaned_mountpoints.svg)

Briefly depicted most important actions:

1.  Loading UniConfig transaction from registry by provided transaction
    ID that is extracted from Cookie header.
2.  Closing connection to database.
3.  Cancellation of database transaction.
4.  Cancellation of datastore read-write transaction.
5.  Unregistration of transaction from local registry.
6.  Unmounting nodes that are not referenced by any UniConfig
    transaction - connection to device is closed and representing
    southbound / Unified mountpoints are removed together with state
    data.

!!!
After transaction is closed, it cannot be used by any other operation - user must create a new transaction in order to use build-and-commit model.
!!!

### Successful example

Closing existing transaction using close-transaction RPC. Response
doesn't body, only status code 200.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:close-transaction' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=2ab7cfc3-dedc-4444-8431-6e9cf94fad3b'
```

```RPC Response, Status: 204
```

### Failed example

If transaction has already been closed, user will receive response with
JSON body containing error message.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:close-transaction' \
--header 'Accept: application/json' \
--header 'Cookie: UNICONFIGTXID=2ab7cfc3-dedc-4444-8431-6e9cf94fad3b'
```

```json RPC Response, Status: 422
{
  "errors": {
    "error": [
      {
        "error-message": "Uniconfig transaction 3819fbaa-6bd4-4c79-bc4e-68f70cf97903 has already been closed",
        "error-tag": "invalid-transaction",
        "error-type": "protocol"
      }
    ]
  }
}
```

## Transaction cleaner

Transaction cleaner is used for automatic closing of transactions that
are open longer then specified timeout value ('transactionIdleTimeOut' 
or 'maxTransactionAge' setting in the configuration). Transaction resets 
her time setting 'transactionIdleTimeOut' after invoking CRUD, RPC operation, 
and is still valid  for time specified in value of setting. This mechanism 
effectively suppresses application-level errors - open transactions are not 
closed at the end of the workflow.

Next sequence diagram describes cleaning process. Referenced diagram
'Close transaction' is placed in the previous 'Closing transaction'
section.

![Transaction cleaner](transaction-cleaner-Transaction_cleaner.svg)

## Use cases

### Modification of different devices in separate transactions

**1. Installation of 2 devices - ‘xr6\_1’ and ‘xr6\_2’ (without
transaction ID)**

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "node-id": "xr6_1",
        "netconf": {
            "netconf-node-topology:host": "192.168.1.211",
            "netconf-node-topology:port": 830,
            "netconf-node-topology:tcp-only": false,
            "netconf-node-topology:username": "cisco",
            "netconf-node-topology:password": "cisco",
            "netconf-node-topology:session-timers": {
                "netconf-node-topology:keepalive-delay": 0
            },
            "netconf-node-topology:other-parameters": {
                "netconf-node-topology:dry-run-journal-size": 180
            },
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:blacklist": {
                "uniconfig-config:path": [
                    "openconfig-interfaces:interfaces",
                    "ietf-interfaces:interfaces",
                    "openconfig-vlan:vlans",
                    "openconfig-routing-policy:routing-policy"
                ]
            }
        }
    }
}'

curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "node-id": "xr6_2",
        "netconf": {
            "netconf-node-topology:host": "192.168.1.212",
            "netconf-node-topology:port": 830,
            "netconf-node-topology:tcp-only": false,
            "netconf-node-topology:username": "cisco",
            "netconf-node-topology:password": "cisco",
            "netconf-node-topology:session-timers": {
                "netconf-node-topology:keepalive-delay": 0
            },
            "netconf-node-topology:other-parameters": {
                "netconf-node-topology:dry-run-journal-size": 180
            }
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:blacklist": {
                "uniconfig-config:path": [
                    "openconfig-interfaces:interfaces",
                    "ietf-interfaces:interfaces",
                    "openconfig-vlan:vlans",
                    "openconfig-routing-policy:routing-policy"
                ]
            }
        }
    }
}'
```

**2. Creation of 2 uniconfig transactions: let’s name them TX1 and TX2**

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node01v71oyfxlkbty1o7f0neiioc0j0.node0'
```

The first response contains transaction-id of TX1 that can be used in
the subsequent requests that belong to TX1:

```text RPC Response, Status: 201
Set cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc;Version=1;Comment="uniconfig transaction created";Path=/rests/
```

The first second contains transaction-id of TX2 that can be used in the
subsequent requests that belong to TX2:

```text RPC Response, Status: 201
Set cookie: UNICONFIGTXID=5e8ab9d0-803a-40d6-9f0a-92e47524bab8;Version=1;Comment="uniconfig transaction created";Path=/rests/
```

**3. Modification of ‘xr6\_1’ uniconfig configuration inside TX1**

Creation of new Loopback97 interface in the TX1 - cookie header contains
UNICONFIGTXID of TX1:

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc; JSESSIONID=node0dsw5vbjrst7m1w2htold3gf6c2.node0' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "act",
            "interface-name": "Loopback97",
            "description": "stanov loopback",
            "interface-virtual": [
                null
            ]
        }
    ]
}'
```

Response:

```RPC Response, Status: 201
```

Verification if TX1 contains created interface (Cookie header contains
UNICONFIGTXID of TX1):

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97?content=config' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc; JSESSIONID=node0dsw5vbjrst7m1w2htold3gf6c2.node0'
```

Response:

```json RPC Response, Status: 200
{
  "interface-configuration": [
    {
      "active": "act",
      "interface-name": "Loopback97",
      "description": "stanov loopback",
      "interface-virtual": [
        null
      ]
    }
  ]
}
```

**4. Modification of ‘xr6\_2’ uniconfig configuration inside TX2**

Creation of new Loopback79 interface - cookie header contains
UNICONFIGTXID of TX2:

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5e8ab9d0-803a-40d6-9f0a-92e47524bab8; JSESSIONID=node0ipnsppvl6r81psxzqbazaoj33.node0' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "act",
            "interface-name": "Loopback79",
            "description": "stanov loopback",
            "interface-virtual": [
                null
            ]
        }
    ]
}'
```

Response:

```RPC Response Status: 201
```

Verification if TX2 contains created interface (Cookie header contains
UNICONFIGTXID of TX2):

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79?content=config' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5e8ab9d0-803a-40d6-9f0a-92e47524bab8; JSESSIONID=node0ipnsppvl6r81psxzqbazaoj33.node0'
```

Response:

```json RPC Response, Status: 200
{
  "interface-configuration": [
    {
      "active": "act",
      "interface-name": "Loopback79",
      "description": "stanov loopback",
      "interface-virtual": [
        null
      ]
    }
  ]
}
```

**5. Verification if TX1 and TX2 are isolated**

TX1 doesn't see modifications done in TX2 and vice-versa:

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5e8ab9d0-803a-40d6-9f0a-92e47524bab8; JSESSIONID=node01ojoxzbr05hlopwixblbfbk7i5.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc; JSESSIONID=node01ojoxzbr05hlopwixblbfbk7i5.node0'
```

Both responses should return Status 404 Not Found:

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-message": "Request could not be completed because the relevant data model content does not exist",
        "error-tag": "data-missing",
        "error-type": "protocol"
      }
    ]
  }
}
```

**6. Committing TX1 and TX2 using uniconfig-manager:commit RPC**

It is not required to specify target nodes in the input because
UniConfig transaction tracks modified nodes:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc; JSESSIONID=node01ojoxzbr05hlopwixblbfbk7i5.node0' \
--data-raw '{
    "input": {
    }
}'
```

Since there aren't any conflicts between modifications in the committed
transactions, both RPCs should succeed. Expected responses:

```RPC Response, Status: 204
```

```RPC Response, Status: 204
```

**7. Verification of committed data**

Verification if configuration was correctly committed to devices (direct
read under yang-ext:mount) and if datastore was updated (GET request
without transaction ID):

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79?content=config' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0jlfqlo2ggxgxe9fv0y6y5ogh7.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79?content=nonconfig' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0jlfqlo2ggxgxe9fv0y6y5ogh7.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=xr6_2/yang-ext:mount/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0jlfqlo2ggxgxe9fv0y6y5ogh7.node0'
```

All 3 responses - Status 200 OK with returned expected data. Similar
verification can be done on 'xr6\_2'.

**8. Verification if TX1 and TX2 are closed**

Trying to read some data in the TX1:

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5e8ab9d0-803a-40d6-9f0a-92e47524bab8; JSESSIONID=node0jlfqlo2ggxgxe9fv0y6y5ogh7.node0'
```

Response - Status 422 Unprocessable Entity:

```json RPC Response, Status: 422
{
  "errors": {
    "error": [
      {
        "error-message": "Uniconfig transaction 5e8ab9d0-803a-40d6-9f0a-92e47524bab8 has already been closed",
        "error-tag": "invalid-transaction",
        "error-type": "protocol"
      }
    ]
  }
}
```

Trying to read some data in the TX2:

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_2/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=73f85310-a20a-46b9-beaf-d2ac98cc74cc; JSESSIONID=node0jlfqlo2ggxgxe9fv0y6y5ogh7.node0'
```

Response - Status 422 Unprocessable Entity:

```json RPC Response, Status: 422
{
  "errors": {
    "error": [
      {
        "error-message": "Uniconfig transaction 73f85310-a20a-46b9-beaf-d2ac98cc74cc has already been closed",
        "error-tag": "invalid-transaction",
        "error-type": "protocol"
      }
    ]
  }
}
```

### Modification of sub-tree on same device in separate transactions

**1. Installation of device ‘xr6\_1’**

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "node-id": "xr6_1",
        "netconf": {
            "netconf-node-topology:host": "192.168.1.211",
            "netconf-node-topology:port": 830,
            "netconf-node-topology:tcp-only": false,
            "netconf-node-topology:username": "cisco",
            "netconf-node-topology:password": "cisco",
            "netconf-node-topology:session-timers": {
                "netconf-node-topology:keepalive-delay": 0
            },
            "netconf-node-topology:other-parameters": {
                "netconf-node-topology:dry-run-journal-size": 180
            }
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:blacklist": {
                "uniconfig-config:path": [
                    "openconfig-interfaces:interfaces",
                    "ietf-interfaces:interfaces",
                    "openconfig-vlan:vlans",
                    "openconfig-routing-policy:routing-policy"
                ]
            }
        }
    }
}'
```

**2. Preparation of configuration on 'xr6\_1'**

Creation of Loopback97 interface with some initial description:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=4b44231a-b2e5-457d-9d88-8f970d33201e'

curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=4b44231a-b2e5-457d-9d88-8f970d33201e; JSESSIONID=node0wbvwgmd3tbx8uu6sw13rcfq210.node0' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "act",
            "interface-name": "Loopback97",
            "description": "stanov loopback",
            "interface-virtual": [
                null
            ]
        }
    ]
}'

curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=4b44231a-b2e5-457d-9d88-8f970d33201e; JSESSIONID=node0wbvwgmd3tbx8uu6sw13rcfq210.node0' \
--data-raw '{
    "input": {
    }
}'
```

**3. Creation of 2 uniconfig transactions: let’s name them TX1 and TX2**

Creation of the uniconfig transaction TX1:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node01l0nvy00n2jeuuhtneqqlhra811.node0'
```

Creation of the uniconfig transaction TX2:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:create-transaction' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node01l0nvy00n2jeuuhtneqqlhra811.node0'
```

**4. Modification of ‘xr6\_1’ uniconfig configuration inside TX1**

Changing description of interface Loopback97 to 'test loopback':

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=14804ad7-c0f6-4b22-8336-7484efc23e31; JSESSIONID=node01l0nvy00n2jeuuhtneqqlhra811.node0' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "act",
            "interface-name": "Loopback97",
            "description": "test loopback",
            "interface-virtual": [
                null
            ]
        }
    ]
}'
```

**5. Modification of ‘xr6\_1’ uniconfig configuration inside TX2**

Changing description of interface Loopback97 to 'next loopback': - there
is a conflict with TX1 which also tries to create/replace the
configuration of the same interface:

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5160a334-aeb0-4864-931a-bc652990feb0; JSESSIONID=node0hnl6qz34dx7b1v97p07vwfqk213.node0' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "act",
            "interface-name": "Loopback97",
            "description": "next loopback",
            "interface-virtual": [
                null
            ]
        }
    ]
}'
```

**6. Commit TX1**

Commit TX2 without target nodes - it should pass:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5160a334-aeb0-4864-931a-bc652990feb0; JSESSIONID=node0nqbpx1l07fbilbrxdj1h0vx15.node0' \
--data-raw '{
    "input": {
    }
}'
```

Response:

```RPC Response, Status: 204
```

**7. Commit TX2**

Commit TX1 without target nodes - it should fail because the same node
has already been modified by different transaction that has already been
committed:

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=14804ad7-c0f6-4b22-8336-7484efc23e31; JSESSIONID=node0nqbpx1l07fbilbrxdj1h0vx15.node0' \
--data-raw '{
    "input": {
    }
}'
```

Response - Status 500 with error message:

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-tag": "validation-failed",
        "error-type": "application",
        "error-message": "TransactionCommitFailedException{message=\"14804ad7-c0f6-4b22-8336-7484efc23e31: Node 'xr6_1' in topology 'uniconfig' has been modified by other transactions."
      }
    ]
  }
}
```

**8. Verification of committed data in TX1 / non-committed data in TX2**

Verification if committed changes in TX1 were applied to datastore and
device:

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97?content=config' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0ovyof0o58vgp5868y4028wd216.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97?content=nonconfig' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0cxljma02pmdb1g6w7ngnyiuwd17.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=xr6_1/yang-ext:mount/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: JSESSIONID=node0cxljma02pmdb1g6w7ngnyiuwd17.node0'

Response for all three requests (description of interface is 'test loopback' - TX2 hasn't been committed):
```

```json RPC Response, Status: 200
{
  "interface-configuration": [
    {
      "active": "act",
      "interface-name": "Loopback97",
      "description": "test loopback",
      "interface-virtual": [
        null
      ]
    }
  ]
}
```

**9. Verification if TX1 and TX2 are closed**

Trying to read some data in the transaction:

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback97' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=5160a334-aeb0-4864-931a-bc652990feb0; JSESSIONID=node0cxljma02pmdb1g6w7ngnyiuwd17.node0'

curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=xr6_1/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Loopback79' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Cookie: UNICONFIGTXID=14804ad7-c0f6-4b22-8336-7484efc23e31; JSESSIONID=node0cxljma02pmdb1g6w7ngnyiuwd17.node0'
```

Respective responses:

```json RPC Response, Status: 403
{
  "errors": {
    "error": [
      {
        "error-message": "Uniconfig transaction 5160a334-aeb0-4864-931a-bc652990feb0 has already been closed",
        "error-tag": "access-denied",
        "error-type": "protocol"
      }
    ]
  }
}
```

```json RPC Response, Status: 403
{
  "errors": {
    "error": [
      {
        "error-message": "Uniconfig transaction 14804ad7-c0f6-4b22-8336-7484efc23e31 has already been closed",
        "error-tag": "access-denied",
        "error-type": "protocol"
      }
    ]
  }
}
```
