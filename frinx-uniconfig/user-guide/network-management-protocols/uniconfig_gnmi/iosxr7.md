# IosXR 7 device

## Install device

A IosXR7 device can be installed through gNMI with the following request:

```bash
curl --location 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Content-Type: application/json' \
--data '{
    "input": {
        "node-id": "iosxr7",
        "gnmi": {
            "schema-cache-directory": "{cache-directory}",
            "uniconfig-config:whitelist": {
                "path": [
                    "Cisco-IOS-XR-ifmgr-cfg:interface-configurations",
                    "openconfig-interfaces:interfaces"
                ]
            },
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:sequence-read-active": true,
            "connection-parameters": {
                "host": "127.0.0.1",
                "port": 57400,
                "connection-type": "INSECURE",
                "device-type": "iosxr7",
                "credentials": {
                    "username": "admin",
                    "password": "admin"
                }
            },
            "session-timers": {
                "request-timeout": 100
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
        "node-id": "iosxr7",
        "connection-type": "gnmi"
    }
}'
```