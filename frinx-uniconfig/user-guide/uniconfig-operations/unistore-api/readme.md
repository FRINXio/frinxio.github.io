# UniStore API

UniStore nodes are used to store and manage various settings and configurations
inside UniConfig. The difference between UniStore and UniConfig nodes is that
UniConfig nodes are backed by a real/network device, whereas UniStore nodes do
not correspond to real devices. In the case of UniStore nodes, UniConfig is
used only to manage the configuration and its persistence into the PostgreSQL
DBMS.

UniStore nodes have the following characteristics:

- UniStore nodes are not backed by "real" devices or southbound mountpoints.
    They are used only to store configurations which are committed to the
    PostgreSQL DBMS.
- The configuration of a UniStore node can be read, created, removed and updated
    in the same way as UniConfig topology nodes. You can use the the same set of
    CRUD RESTCONF operations and supported UniConfig RPCs for operation
    purposes.
- UniStore nodes are placed in a dedicated `unistore` topology under
    network-topology nodes. The whole configuration is placed under the
    `configuration` container.
- The UniStore configuration is modelled by user-provided YANG schemas that can
    be loaded into UniConfig. When creating a new UniStore node, you must
    provide the name of a YANG repository so that UniConfig knows how to parse
    the configuration (`uniconfig-schema-repository` query parameter).

UniConfig operations that are supported for UniStore nodes:

- All RESTCONF CRUD operations
- commit / checked-commit RPC
- calculate-diff RPC (including git-like-diff flavor)
- subtree-manager RPCs
- replace-config-with-oper RPC
- revert-changes RPC (transaction-log feature)

!!!
The node ID of a UniStore node must be unique among all UniConfig and UniStore
nodes.
!!!

## Commit operation

Actions performed with UniStore nodes during a commit operation:

1. Verify configuration fingerprint. If another UniConfig transaction has
    already changed one of the UniStore nodes touched in the current
    transaction, the commit operation fails.
2. Calculate diff operation across all changed UniStore nodes.
3. Write intended configuration into the UniConfig transaction.
4. Replace the actual configuration with the intended configuration in the
   UniConfig transaction.
5. Update last configuration fingerprint to the UUID of the committed
    transaction.
6. Write transaction log into transaction.
7. Commit UniConfig transaction. Cached changes are sent to the PostgreSQL DBMS.

## Example use case

### Prepare YANG repository

A YANG repository is required for UniConfig to model the UniStore node
configuration. A UniStore node can only be modeled by one YANG repository.

To provide a YANG repository to UniConfig, copy the directory containing the
YANG files under the **cache** parent directory. From there, it is loaded either
at startup or in runtime with the **register-repository RPC**.

As an example, assume that **cache** contains the **system** YANG repository with a
simple YANG module:

```yang config@2021-09-30.yang:
module config {
    yang-version 1.1;
    namespace urn:ietf:params:xml:ns:yang:test;
    prefix test;

    revision 2021-09-30 {
        description
          "Initial revision";
    }

    container settings {
        leaf update-interval {
            type uint16;
        }

        container cluster {
            leaf ping-interval {
                type uint16;
            }
            leaf idle-timeout {
                type uint16;
            }
        }

        list routing-protocols {
            key process-id;

            leaf process-id {
                type string;
            }

            leaf-list interfaces {
                type string;
            }
        }
    }
}
```

### Create UniStore node

The following request shows how a new Unistore node (`global`) is created using
the provided JSON payload and the name of the YANG repository used to parse the
payload (query parameter `uniconfig-schema-repository`). Note that the YANG
repository must be specified only when the UniStore node is initialized.

```bash Creation of UniStore node with ID ‘global’
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global/configuration/settings?uniconfig-schema-repository=system' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "settings": {
        "update-interval": 5000,
        "cluster": {
            "ping-interval": 20000,
            "idle-timeout": 100
        },
        "routing-protocols": [
            {
                "process-id": "rip",
                "interfaces": [
                    "eth0.0",
                    "eth0.1"
                ]
            },
            {
                "process-id": "ospf",
                "interfaces": [
                    "eth0.1"
                ]
            }
        ]
    }
}'
```

```json RPC Response, Status: 201

```

### Read content of UniStore node

The following sample shows how to read the content of a UniStore node using a
regular `GET` request. The query parameter `content` is set to `config`,
indicating that the UniStore node is cached only in the `Configuration`
datastore of the transaction (the `Operational` datastore is empty at this
time).

```bash Reading UniStore node with ID ‘global’
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global/configuration?content=config' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
    "frinx-uniconfig-topology:configuration": {
        "config:settings": {
            "update-interval": 5000,
            "cluster": {
                "ping-interval": 20000,
                "idle-timeout": 100
            },
            "routing-protocols": [
                {
                    "process-id": "rip",
                    "interfaces": [
                        "eth0.0",
                        "eth0.1"
                    ]
                },
                {
                    "process-id": "ospf",
                    "interfaces": [
                        "eth0.1"
                    ]
                }
            ]
        }
    }
}
```

### Calculate-diff RPC (created node)

The calculate-diff operation is also supported for UniStore nodes. The following
request shows the diff for all touched nodes in the current transaction,
including UniStore nodes. As the UniStore node was just created, the diff output
only contains `created-data` with the entire root `settings` container.

```bash Calculate-diff RPC
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": []
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
          "node-id": "global",
          "topology-id": "unistore",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=global/frinx-uniconfig-topology:configuration/config:settings",
              "data": "{\n  \"config:settings\": {\n    \"update-interval\": 5000,\n    \"cluster\": {\n      \"ping-interval\": 20000,\n      \"idle-timeout\": 100\n    },\n    \"routing-protocols\": [\n      {\n        \"process-id\": \"rip\",\n        \"interfaces\": [\n          \"eth0.0\",\n          \"eth0.1\"\n        ]\n      },\n      {\n        \"process-id\": \"ospf\",\n        \"interfaces\": [\n          \"eth0.1\"\n        ]\n      }\n    ]\n  }\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Persistence of UniStore nodes

For UniStore nodes, the **commit RPC** is used to confirm the changes that have
been made and to store them in the PostgreSQL DBMS. As described in the previous
section, the commit operation stores the UniStore node configuration and
transaction log in the DBMS and does not touch network devices.

```bash Commit RPC
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```RPC Response, Status: 204
```

!!!
Changes to UniStore and UniConfig nodes can be combined in the same transaction
and can be committed simultaneously.
!!!

### Read committed configuration

Since the configuration was committed in the previous step, it is also visible
in the `Operational` datastore of the newly created transaction. The actual
state can be read by appending the `content=nonconfig` query parameter to a
`GET` request, as shown in the following example:

```bash Reading UniStore node with ID **'global'**
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global/configuration?content=nonconfig' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
  "frinx-uniconfig-topology:configuration": {
    "config:settings": {
      "update-interval": 5000,
      "cluster": {
        "ping-interval": 20000,
        "idle-timeout": 100
      },
      "routing-protocols": [
        {
          "process-id": "rip",
          "interfaces": [
            "eth0.0",
            "eth0.1"
          ]
        },
        {
          "process-id": "ospf",
          "interfaces": [
            "eth0.1"
          ]
        }
      ]
    }
  }
}
```

### Verify configuration fingerprint

The configuration fingerprint is part of the optimistic locking mechanism. By
comparing fingerprints from the beginning of the transaction and from the commit
operation, it is possible to see if another UniConfig transaction has already
modified the affected UniStore node. For Unistore nodes, the fingerprint is
always updated to match the transaction id (UUID) of the last committed
transaction containing the UniStore node.

```bash Reading UniStore node with ID **'global'**
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global/configuration-metadata/last-configuration-fingerprint?content=nonconfig' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
  "frinx-configuration-metadata:last-configuration-fingerprint": "428d1a55-5681-4a47-8a51-634a215127d7"
}
```

### Modify configuration

The same RESTCONF CRUD operations that can be applied to UniConfig nodes are
also relevant to UniStore nodes. The following request demonstrates how to merge
multiple fields using the `PATCH` operation.

```bash Merging configuration
curl --location --request PATCH 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global/configuration/settings' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "settings": {
        "cluster": {
            "ping-interval": 10000
        },
        "routing-protocols": [
            {
                "process-id": "ospf",
                "interfaces": [
                    "eth0.1",
                    "eth0.2"
                ]
            },
            {
                "process-id": "eigrp",
                "interfaces": [
                    "g1"
                ]
            }
        ]
    }
}'
```

```RPC Response, Status: 200
```

### Calculate-diff RPC (updated node)

The second calculate-diff RPC shows more granular changes made to an
existing UniStore node, which includes the `create-data` and `updated-data`
entries.

```bash Calculate-diff RPC
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": []
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
          "node-id": "global",
          "topology-id": "unistore",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=global/frinx-uniconfig-topology:configuration/config:settings/routing-protocols=eigrp",
              "data": "{\n  \"routing-protocols\": [\n    {\n      \"process-id\": \"eigrp\",\n      \"interfaces\": [\n        \"g1\"\n      ]\n    }\n  ]\n}"
            }
          ],
          "updated-data": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=global/frinx-uniconfig-topology:configuration/config:settings/routing-protocols=ospf",
              "data-actual": "{\n  \"config:interfaces\": [\n    \"eth0.1\"\n  ]\n}",
              "data-intended": "{\n  \"config:interfaces\": [\n    \"eth0.1\",\n    \"eth0.2\"\n  ]\n}"
            },
            {
              "path": "/network-topology:network-topology/topology=unistore/node=global/frinx-uniconfig-topology:configuration/config:settings/cluster/ping-interval",
              "data-actual": "{\n  \"config:ping-interval\": 20000\n}",
              "data-intended": "{\n  \"config:ping-interval\": 10000\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Display content of transaction log

Committed transactions, including all metadata such as serialized diff output or
transaction ID, can be displayed by reading the `transactions-metadata`
container in the `Operational` datastore. Information about successfully
committed UniStore nodes is also displayed. This can be used to revert changes
using the transaction ID displayed in the transaction log.

```bash Reading entries of transaction-log
curl --location --request GET 'http://localhost:8181/rests/data/transactions-metadata/transaction-metadata?content=nonconfig' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
  "transaction-metadata": [
    {
      "transaction-id": "428d1a55-5681-4a47-8a51-634a215127d7",
      "username": "admin",
      "metadata": [
        {
          "node-id": "global",
          "diff": [
            {
              "path": "/config:settings",
              "data-after": "{\"config:settings\": {\"cluster\": {\"idle-timeout\": 100, \"ping-interval\": 20000}, \"update-interval\": 5000, \"routing-protocols\": [{\"interfaces\": [\"eth0.0\", \"eth0.1\"], \"process-id\": \"rip\"}, {\"interfaces\": [\"eth0.1\"], \"process-id\": \"ospf\"}]}}"
            }
          ],
          "topology": "unistore"
        }
      ],
      "commit-time": "2021-Oct-10 18:52:40.942 +0200"
    },
    {
      "transaction-id": "107e38d9-fa85-4bd7-89f3-76beac55345a",
      "username": "admin",
      "metadata": [
        {
          "node-id": "global",
          "diff": [
            {
              "path": "/config:settings/routing-protocols=eigrp",
              "data-after": "{\"routing-protocols\": [{\"interfaces\": [\"g1\"], \"process-id\": \"eigrp\"}]}"
            },
            {
              "path": "/config:settings/cluster/ping-interval",
              "data-before": "{\"config:ping-interval\": 20000}",
              "data-after": "{\"config:ping-interval\": 10000}"
            },
            {
              "path": "/config:settings/routing-protocols=ospf",
              "data-before": "{\"config:interfaces\": [\"eth0.1\"]}",
              "data-after": "{\"config:interfaces\": [\"eth0.1\", \"eth0.2\"]}"
            }
          ],
          "topology": "unistore"
        }
      ],
      "commit-time": "2021-Oct-10 19:12:34.241 +0200"
    }
  ]
}
```

### Remove UniStore node

A UniStore node can be removed either by sending a `DELETE` request to the
entire `node` list entry or `configuration` container, or by removing all
`configuration` child entities. In each case, the UniStore node is removed after
changes are confirmed using the **commit RPC**.

```bash Removal of UniStore node
curl --location --request DELETE 'http://localhost:8181/rests/data/network-topology:network-topology/topology=unistore/node=global' \
--header 'Accept: application/json'
```

```RPC Response, Status: 204
```

```bash Commit RPC
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```RPC Response, Status: 204
```
