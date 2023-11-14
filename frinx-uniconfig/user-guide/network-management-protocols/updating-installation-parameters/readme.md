---
label: Updating installation parameters
order: 8000
---

# Updating installation parameters

During device installation, UniConfig creates a mountpoint for the device and stores it in the database. The mountpoint contains all parameters set in the installation request.

UniConfig includes a feature to update mountpoint parameters, which can be used for NETCONF, CLI and gNMI nodes.

### Show installation parameters

Parameters for installed devices can be shown using a `GET` request on the node, which returns the current node settings. Make sure to specify the right topology. See below for examples.

!!!
By default, the `NETCONF`, `CLI` and `gNMI` topologies have the password parameter encrypted. This can be changed
in the corresponding YANG schema by adding or removing the `frinx-encrypt:encrypt` extension flag.
!!!

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
            "uniconfig-config:install-uniconfig-node-enabled": true,
            "cli-topology:host": "192.168.1.225",
            "cli-topology:transport-type": "ssh",
            "cli-topology:dry-run-journal-size": 150,
            "cli-topology:username": "test",
            "cli-topology:password": "rsa_lfOgcqEIl/d2E5SxZ5xFJY0hzwrauUu6l...",
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
            "netconf-node-topology:tcp-only": false,
            "netconf-node-topology:username": "test",
            "netconf-node-topology:password": "rsa_lfOgcqEIl/d2E5SxZ5xFJY0hzwrauUu6l...",
            "netconf-node-topology:session-timers" : {
                "netconf-node-topology:keepalive-delay": 0
            },
            "netconf-node-topology:other-parameters" : {
                "netconf-node-topology:edit-config-test-option": "test-then-set"
            },
            "uniconfig-config:uniconfig-native-enabled": true,
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

**gNMI node**

```bash
curl -X GET \
  http://127.0.0.1:8181/rests/data/network-topology:network-topology/topology=gnmi-topology/node=gnmiNode
```

Output:
```
{
    "node": [
        {
            "node-id": "sonic",
            "uniconfig-config:sequence-read-active": true,
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:whitelist": {
                "path": [
                    "openconfig-interfaces:interfaces",
                    "openconfig-network-instance:network-instances",
                    "openconfig-relay-agent:relay-agent",
                    "openconfig-port-group:port-groups",
                    "openconfig-mclag:mclag",
                    "openconfig-lldp:lldp",
                    "sonic-vlan:sonic-vlan",
                    "openconfig-platform:components",
                    "openconfig-system:system",
                    "openconfig-neighbor:neighbor-globals",
                    "sonic-mclag:sonic-mclag"
                ]
            },
            "uniconfig-config:admin-state": "unlocked",
            "gnmi-topology:schema-cache-directory": "gnmi-topology-v400-7",
            "gnmi-topology:extensions-parameters": {
                "gnmi-parameters": {
                    "use-model-name-prefix": true
                },
                "gnmi-force-capabilities:force-cached-capabilities": [
                    null
                ]
            },
            "gnmi-topology:session-timers": {
                "request-timeout": 180
            },
            "gnmi-topology:connection-parameters": {
                "port": 8080,
                "credentials": {
                    "username": "<username>",
                    "password": "rsa_J1H0YDNQSvmGrn5QP+MfBLF9OOzWEaWJElcr7K5ew7enUr...."
                },
                "device-type": "sonic",
                "host": "10.19.0.252",
                "connection-type": "INSECURE"
            },
        }
    ]
}
```

### Update installation parameters

To update node installation parameters, use a `PUT` request with an updated request body copied from the `GET` request in the previous section. Single parameters can also be updated with a direct `PUT` call to the specific parameter.

If the password parameter is set to be encrypted, changing it will encrypt the input value.

**CLI node**

Update multiple parameters:
- `host`
- `dry-run-journal-size`
- `journal-size`

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=cli/node=cliNode \
  -d 
  '{
     "node": [
        {
            "node-id": "cliNode",
            "uniconfig-config:install-uniconfig-node-enabled": true,
            "cli-topology:host": "192.168.1.230",
            "cli-topology:transport-type": "ssh",
            "cli-topology:dry-run-journal-size": 170,
            "cli-topology:username": "test",
            "cli-topology:password": "test",
            "cli-topology:journal-size": 160,
            "cli-topology:port": 22,
            "cli-topology:device-version": "6.2.3",
            "cli-topology:device-type": "ios xr"
        }
     ]
  }'
```

Update a single parameter:
- `host`

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=cli/node=cliNode/cli-topology:host \
  -d 
  '{
     "cli-topology:host": "192.168.1.230"
   }'
```

**NETCONF node**

Update multiple parameters:
- `host`
- `keepalive-delay`

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=netconfNode \
  -d 
  '{
      "input": {
          "node-id": "netconfNode",
          "netconf": {
              "netconf-node-topology:host": "192.168.1.214",
              "netconf-node-topology:port": 830,
              "netconf-node-topology:tcp-only": false,
              "netconf-node-topology:username": "test",
              "netconf-node-topology:password": "test",
              "uniconfig-config:uniconfig-native-enabled": true,
              "netconf-node-topology:session-timers" : {
                  "netconf-node-topology:keepalive-delay": 5
              },
              "netconf-node-topology:other-parameters" : {
                  "netconf-node-topology:edit-config-test-option": "test-then-set"
              },
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

Update a single parameter:
- `host`

```bash
curl -X PUT \
  http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=netconfNode/netconf-node-topology:host \
  -d 
  '{
     "netconf-node-topology:host": "192.168.1.214"
   }'
```

After these changes have been made, use the `GET` requests in the **Show installation parameters** section above to see that the parameters have actually been changed. You can also use the `GET` request for a single parameter.
