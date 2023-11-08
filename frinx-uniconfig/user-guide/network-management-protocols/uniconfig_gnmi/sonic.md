# SONiC device

## Install SONiC device

SONiC device can be installed through gNMI with the following request:

```bash
curl --location 'http://localhost:8181/rests/operations/connection-manager:install-node' \
--header 'Content-Type: application/json' \
--data '{
    "input": {
        "node-id": "sonic",
        "gnmi": {
            "schema-cache-directory": "gnmi-topology",
            "update-paths": [
                "openconfig-interfaces:interfaces\/interface=.*[Ee]thernet?[0-9]+\/.*",
                "[^\/]+",
                "sonic-vlan:sonic-vlan.*"
            ],
            "replace-paths": [
                "openconfig-interfaces:interfaces/interface=$.*[Ee]thernet?[0-9]+/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/config/trunk-vlans",
                "openconfig-interfaces:interfaces/interface=$.*[Pp]ortChannel?[0-9]+/openconfig-if-aggregate:aggregation/openconfig-vlan:switched-vlan/config/trunk-vlans",
                "openconfig-relay-agent:relay-agent/dhcp/interfaces/interface=$[Vv]lan[0-9]+/config",
                "openconfig-relay-agent:relay-agent/dhcpv6/interfaces/interface=$[Vv]lan[0-9]+/config",
                "openconfig-platform:components/component=$.*/openconfig-platform:port/openconfig-platform-port:breakout-mode"
            ],
            "remove-module-names-paths": [
                "network-instances/network-instance=default/protocols/protocol"
            ],
            "all-type-paths": [
                "openconfig-lldp:lldp",
                "openconfig-port-group:port-groups"
            ],
            "dependency-paths": [
                {
                    "before": "openconfig-network-instance:network-instances/network-instance",
                    "after": "openconfig-network-instance:network-instances/network-instance/interfaces/interface"
                },
                {
                    "before": "openconfig-network-instance:network-instances/network-instance/interfaces/interface",
                    "after": "openconfig-network-instance:network-instances/network-instance/openconfig-vxlan:vxlan-vni-instances/vni-instance"
                },
                {
                    "before": "openconfig-network-instance:network-instances/network-instance/interfaces/interface",
                    "after": "openconfig-interfaces:interfaces/interface/openconfig-vlan:routed-vlan/openconfig-if-ip:ipv4/openconfig-interfaces-ext:sag-ipv4/config"
                },
                {
                    "before": "openconfig-network-instance:network-instances/network-instance/interfaces/interface",
                    "after": "openconfig-interfaces:interfaces/interface/openconfig-vlan:routed-vlan/openconfig-if-ip:ipv6/openconfig-interfaces-ext:sag-ipv6/config"
                },
                {
                    "before": "openconfig-network-instance:network-instances/network-instance/openconfig-vxlan:vxlan-vni-instances/vni-instance",
                    "after": "openconfig-relay-agent:relay-agent/dhcp/interfaces/interface"
                },
                {
                    "before": "openconfig-network-instance:network-instances/network-instance/openconfig-vxlan:vxlan-vni-instances/vni-instance",
                    "after": "openconfig-relay-agent:relay-agent/dhcpv6/interfaces/interface"
                },
                {
                    "before": "openconfig-interfaces:interfaces/interface/openconfig-vlan:routed-vlan/openconfig-if-ip:ipv4/openconfig-interfaces-ext:sag-ipv4/config",
                    "after": "openconfig-relay-agent:relay-agent/dhcp/interfaces/interface"
                },
                {
                    "before": "openconfig-interfaces:interfaces/interface/openconfig-vlan:routed-vlan/openconfig-if-ip:ipv6/openconfig-interfaces-ext:sag-ipv6/config",
                    "after": "openconfig-relay-agent:relay-agent/dhcpv6/interfaces/interface"
                },
                {
                    "before": "openconfig-interfaces:interfaces/interface",
                    "after": "openconfig-mclag:mclag/interfaces/interface"
                },
                {
                    "before": "openconfig-interfaces:interfaces/interface",
                    "after": "openconfig-interfaces:interfaces/interface/openconfig-if-ethernet:ethernet/config/aggregate-id"
                },
                {
                    "before" : "openconfig-interfaces:interfaces/interface",
                    "after": "openconfig-network-instance:network-instances/network-instance/interfaces/interface"
                },
                {
                    "before" : "openconfig-network-instance:network-instances/network-instance",
                    "after": "openconfig-interfaces:interfaces/interface/subinterfaces"
                }
            ],
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
            "uniconfig-config:uniconfig-native-enabled": true,
            "uniconfig-config:sequence-read-active": true,
            "connection-parameters": {
                "host": "<ip>",
                "port": "8080",
                "device-type" : "sonic",
                "connection-type": "INSECURE",
                "credentials": {
                    "username": "<username>",
                    "password": "<password>"
                }
            },
            "session-timers": {
                "request-timeout" : 180
            },
            "other-parameters" : {
                "dry-run-journal-size" : 240
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


To uninstall device run:

```bash
curl --location 'http://localhost:8181/rests/operations/connection-manager:uninstall-node' \
--header 'Content-Type: application/json' \
--data '{
    "input": {
        "node-id": "sonic",
        "connection-type": "gnmi"
    }
}'
```