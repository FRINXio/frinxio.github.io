# Dry-run manager

## RPC dryrun-commit

This RPC resolves the diff between the actual and intended configurationz of
nodes by using the UniConfig Node Manager.

Changes to CLI nodes are applied using the cli-dryrun mountpoint, which only
stores translated CLI commands to the cli-dry-run journal. After all changes are
applied, the cli-dryrun journal is read and an RPC output is created and
returned. This works similarly with NETCONF devices, but produces NETCONF
messages as output instead of CLI commands.

RPC input contains a list of UniConfig nodes for which to execute the dry run.

RPC output describes the results of the operation and matches all input nodes.
It also contains a list of commands and NETCONF messages for the given nodes.

If the RPC is called with an empty list of target nodes, the dryrun operation is
executed on all modified nodes in the UniConfig transaction. If a node fails for
any reason, the entire RPC fails.

![RPC dryrun commit](RPC_dry-run-RPC_dryrun_commit.svg)

## RPC examples

### Successful example

RPC input does not contain the target node. RPC output contains a list of
commands that would be sent to the device if the RPC commit or checked-commit
was called.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "IOSXRN",
          "configuration": "<rpc message-id=\"m-23\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<lock>\n<target>\n<candidate/>\n</target>\n</lock>\n</rpc>\n<rpc message-id=\"m-24\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<get-config>\n<source>\n<candidate/>\n</source>\n<filter xmlns:ns0=\"urn:ietf:params:xml:ns:netconf:base:1.0\" ns0:type=\"subtree\">\n<interface-configurations xmlns=\"http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg\"&gt;\n&lt;interface-configuration&gt;\n&lt;active&gt;act&lt;/active&gt;\n&lt;interface-name&gt;GigabitEthernet0/0/0/1&lt;/interface-name&gt;\n&lt;/interface-configuration&gt;\n&lt;/interface-configurations&gt;\n&lt;/filter&gt;\n&lt;/get-config&gt;\n&lt;/rpc&gt;\n<rpc message-id=\"m-25\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<edit-config>\n<target>\n<candidate/>\n</target>\n<config>\n<interface-configurations xmlns=\"http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg\"&gt;\n&lt;interface-configuration&gt;\n&lt;active&gt;act&lt;/active&gt;\n&lt;interface-name&gt;GigabitEthernet0/0/0/1&lt;/interface-name&gt;\n&lt;mtus/&gt;\n&lt;/interface-configuration&gt;\n&lt;/interface-configurations&gt;\n&lt;/config&gt;\n&lt;/edit-config&gt;\n&lt;/rpc&gt;\n<rpc message-id=\"m-26\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<commit/>\n</rpc>\n<rpc message-id=\"m-27\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<unlock>\n<target>\n<candidate/>\n</target>\n</unlock>\n</rpc>\n<rpc message-id=\"m-28\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<lock>\n<target>\n<candidate/>\n</target>\n</lock>\n</rpc>\n<rpc message-id=\"m-29\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<commit/>\n</rpc>\n<rpc message-id=\"m-30\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<unlock>\n<target>\n<candidate/>\n</target>\n</unlock>\n</rpc>\n"
        },
        {
          "node-id": "IOSXR",
          "configuration": "2019-09-13T09:03:34.072: configure terminal\n2019-09-13T09:03:34.073: interface GigabitEthernet0/0/0/1fghgfhfh\nno shutdown\nroot\n\n2019-09-13T09:03:34.073: commit\n2019-09-13T09:03:34.074: end\n"
        }
      ]
    }
  }
}
```

### Successful example

RPC input does not contain any target nodes. Dryrun is executed with all
modified nodes.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "IOSXRN",
          "configuration": "<rpc message-id=\"m-23\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<lock>\n<target>\n<candidate/>\n</target>\n</lock>\n</rpc>\n<rpc message-id=\"m-24\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<get-config>\n<source>\n<candidate/>\n</source>\n<filter xmlns:ns0=\"urn:ietf:params:xml:ns:netconf:base:1.0\" ns0:type=\"subtree\">\n<interface-configurations xmlns=\"http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg\"&gt;\n&lt;interface-configuration&gt;\n&lt;active&gt;act&lt;/active&gt;\n&lt;interface-name&gt;GigabitEthernet0/0/0/1&lt;/interface-name&gt;\n&lt;/interface-configuration&gt;\n&lt;/interface-configurations&gt;\n&lt;/filter&gt;\n&lt;/get-config&gt;\n&lt;/rpc&gt;\n<rpc message-id=\"m-25\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<edit-config>\n<target>\n<candidate/>\n</target>\n<config>\n<interface-configurations xmlns=\"http://cisco.com/ns/yang/Cisco-IOS-XR-ifmgr-cfg\"&gt;\n&lt;interface-configuration&gt;\n&lt;active&gt;act&lt;/active&gt;\n&lt;interface-name&gt;GigabitEthernet0/0/0/1&lt;/interface-name&gt;\n&lt;mtus/&gt;\n&lt;/interface-configuration&gt;\n&lt;/interface-configurations&gt;\n&lt;/config&gt;\n&lt;/edit-config&gt;\n&lt;/rpc&gt;\n<rpc message-id=\"m-26\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<commit/>\n</rpc>\n<rpc message-id=\"m-27\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<unlock>\n<target>\n<candidate/>\n</target>\n</unlock>\n</rpc>\n<rpc message-id=\"m-28\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<lock>\n<target>\n<candidate/>\n</target>\n</lock>\n</rpc>\n<rpc message-id=\"m-29\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<commit/>\n</rpc>\n<rpc message-id=\"m-30\" xmlns=\"urn:ietf:params:xml:ns:netconf:base:1.0\">\n<unlock>\n<target>\n<candidate/>\n</target>\n</unlock>\n</rpc>\n"
        },
        {
          "node-id": "IOSXR",
          "configuration": "2019-09-13T09:03:34.072: configure terminal\n2019-09-13T09:03:34.073: interface GigabitEthernet0/0/0/1fghgfhfh\nno shutdown\nroot\n\n2019-09-13T09:03:34.073: commit\n2019-09-13T09:03:34.074: end\n"
        }
      ]
    }
  }
}
```

### Successful example

If there are no touched nodes, the request finishes successfully.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {}
}
```

### Failed example

Node R1 has failed due to incorrect configuration.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-failed",
        "error-message": "Supplied value \"GigabitEthernet0/0/0/1ghjfhjfhjfghj\" does not match required pattern \"^(([a-zA-Z0-9_]*\\d+/){3,4}\\d+)|(([a-zA-Z0-9_]*\\d+/){3,4}\\d+\\.\\d+)|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]*\\d+))|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]+))|([a-zA-Z0-9_-]*\\d+)|([a-zA-Z0-9_-]*\\d+\\.\\d+)|(mpls)|(dwdm)$\"",
        "error-info": {
          "node-id": "R1",
          "configuration-status": "fail",
          "rollback-status": "complete"
        }
      }
    ]
  }
}
```

### Failed example

Node R1 has failed because it does not support dry-run functionality.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 501
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-not-supported",
        "error-message": "Node does not support dry-run",
        "error-info": {
          "node-id": "R1"
        }
      }
    ]
  }
}
```

### Failed example

Node R1 has lost connection.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/dryrun-manager:dryrun-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 502
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "southbound_no_connection",
        "error-message": "Unified Mountpoint not found.",
        "error-info": {
          "node-id": "R1",
          "configuration-status": "fail"
        }
      }
    ]
  }
}
```
