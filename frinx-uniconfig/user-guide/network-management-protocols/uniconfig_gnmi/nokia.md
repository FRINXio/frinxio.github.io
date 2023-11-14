# NOKIA device

## Install device

A Nokia device can be installed through gNMI with the following request:

```bash
curl --location 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Content-Type: application/json' \
--data '{
    "input": {
        "node-id": "nokia",
        "gnmi": {
            "uniconfig-config:whitelist": {
                "path": [
                    "openconfig-interfaces:interfaces",
                    "openconfig-network-instance:network-instances",
                    "openconfig-relay-agent:relay-agent",
                    "openconfig-port-group:port-groups",
                    "openconfig-mclag:mclag",
                    "openconfig-lldp:lldp"
                ]
            },
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:sequence-read-active": true,
            "connection-parameters": {
                "host": "127.0.0.1",
                "port": 57400,
                "connection-type": "PLAINTEXT",
                "credentials": {
                    "username": "<username>",
                    "password": "<password>"
                }
            },
            "session-timers": {
                "request-timeout": 100,
            },
            "extensions-parameters": {
                "gnmi-parameters": {
                    "use-model-name-prefix": true
                },
                "force-cached-capabilities": [
                    null
                ]
            }
        }
    }
}'
```

## Uninstall device

To uninstall a device:

```bash
curl --location 'http://localhost:8181/rests/operations/connection-manager:uninstall-node' \
--header 'Content-Type: application/json' \
--data '{
    "input": {
        "node-id": "nokia",
        "connection-type": "gnmi"
    }
}'
```