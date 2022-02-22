---
label: Updating installation parameters
order: 8000
---

# Updating installation parameters

## Overview

During device installation UniConfig creates a mount-point for this device and stores it in the database. This
mount-point contains all parameters set by the user in the installation request. UniConfig supports a feature to update
mount-point parameters. It is possible to use for topology-netconf and also for cli.

### Show installation parameters

Parameters of the installed devices can be displayed by a GET request on the node. It is necessary to use the right
topology. It should return the current node settings. See the following examples:

**CLI node**

```bash
curl -X GET \
  http://127.0.0.1:8181/rests/data/network-topology:network-topology/topology=cli/node=cliNode
```

Output:

```
{
    "node": [
        {
            "node-id": "cliNode",
            "node-extension:reconcile": false,
            "uniconfig-config:install-uniconfig-node-enabled": true,
            "cli-topology:host": "192.168.1.225",
            "cli-topology:transport-type": "ssh",
            "cli-topology:dry-run-journal-size": 150,
            "cli-topology:username": "cisco",
            "cli-topology:password": "cisco",
            "cli-topology:journal-size": 150,
            "cli-topology:port": 22,
            "cli-topology:device-version": "6.2.3",
            "cli-topology:device-type": "ios xr"
        }
    ]
}
```

**NETCONF node**

```bash
curl -X GET \
  http://127.0.0.1:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=netconfNode
```

Output:

```
{
    "input": {
        "node-id": "netconfNode",
        "netconf": {
            "netconf-node-topology:host": "192.168.1.216",
            "netconf-node-topology:port": 830,
            "netconf-node-topology:keepalive-delay": 0,
            "netconf-node-topology:tcp-only": false,
            "netconf-node-topology:username": "cisco",
            "netconf-node-topology:password": "cisco",
            "uniconfig-config:uniconfig-native-enabled": true,
            "netconf-node-topology:edit-config-test-option": "test-then-set",
            "uniconfig-config:blacklist": {
                "uniconfig-config:path": [
                    "openconfig-interfaces:interfaces", "ietf-interfaces:interfaces", "openconfig-vlan:vlans", 
                    "openconfig-routing-policy:routing-policy", "openconfig-terminal-device:terminal-device"
                ]
            }
        }
    }
}
```

### Update installation parameters

To update node installation parameters is possible to use a PUT request with updated request body that is possible to
copy from the GET request from the previous section.

**CLI node**

In this example, the "cli-topology:dry-run-journal-size" parameter is changed from value 150 to 160.

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=cli/node=cliNode \
  -d '{
     "node": [
        {
            "node-id": "cliNode",
            "node-extension:reconcile": false,
            "uniconfig-config:install-uniconfig-node-enabled": true,
            "cli-topology:host": "192.168.1.225",
            "cli-topology:transport-type": "ssh",
            "cli-topology:dry-run-journal-size": 160,
            "cli-topology:username": "cisco",
            "cli-topology:password": "cisco",
            "cli-topology:journal-size": 150,
            "cli-topology:port": 22,
            "cli-topology:device-version": "6.2.3",
            "cli-topology:device-type": "ios xr"
        }
     ]
  }'
```

**NETCONF node**

In this example, the "netconf-node-topology:host" parameter is changed from value 192.168.1.216 to 192.168.1.214.

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=netconfNode \
  -d '{
      "input": {
          "node-id": "netconfNode",
          "netconf": {
              "netconf-node-topology:host": "192.168.1.214",
              "netconf-node-topology:port": 830,
              "netconf-node-topology:keepalive-delay": 0,
              "netconf-node-topology:tcp-only": false,
              "netconf-node-topology:username": "cisco",
              "netconf-node-topology:password": "cisco",
              "uniconfig-config:uniconfig-native-enabled": true,
              "netconf-node-topology:edit-config-test-option": "test-then-set",
              "uniconfig-config:blacklist": {
                  "uniconfig-config:path": [
                      "openconfig-interfaces:interfaces", "ietf-interfaces:interfaces", "openconfig-vlan:vlans", 
                      "openconfig-routing-policy:routing-policy", "openconfig-terminal-device:terminal-device"
                  ]
              }
          }
      }
  }'
```

After these changes, when we use the GET requests from the "Show installation parameters" section, then we can see that 
the parameters have actually been changed.