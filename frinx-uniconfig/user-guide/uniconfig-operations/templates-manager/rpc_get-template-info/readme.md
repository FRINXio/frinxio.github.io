# RPC get-template-info

This RPC shows information about all variables in the specified template.
The RPC input must contain a template name.

## RPC examples

### Creating a template

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=%7B%24act-var%7D,%7B%24ifc-name-var%7D?uniconfig-schema-repository=schemas_1' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "interface-configuration": [
        {
            "active": "{$act-var}",
            "interface-name": "{$ifc-name-var}",
            "shutdown": [
                "{$shutdown-var}"
            ]
        }
    ]
}'
```

```RPC Response, Status: 201
```

### Successful example

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/template-manager:get-template-info' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-node-id": "interface_template"
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "variables": {
      "variable": [
        {
          "variable-name": "{$act-var}",
          "type": "Cisco-IOS-XR-ifmgr-cfg:Interface-active",
          "paths": {
            "path": [
              {
                "path": "/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration={%24act-var},{%24ifc-name-var}/active",
                "path-description": "Whether the interface is active or\npreconfigured"
              }
            ]
          },
          "base-types": {
            "base-type": [
              {
                "constraints": [
                  "Length: [[0..2147483647]]",
                  "Pattern: (act)|(pre)"
                ],
                "type": "string"
              }
            ]
          },
          "type-description": "act:The interface is active, pre:Preconfiguration"
        },
        {
          "variable-name": "{$ifc-name-var}",
          "type": "Cisco-IOS-XR-types:Interface-name",
          "paths": {
            "path": [
              {
                "path": "/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration={%24act-var},{%24ifc-name-var}/interface-name",
                "path-description": "The name of the interface"
              }
            ]
          },
          "base-types": {
            "base-type": [
              {
                "constraints": [
                  "Length: [[0..2147483647]]",
                  "Pattern: (([a-zA-Z0-9_]*\\d+/){3}\\d+)|(([a-zA-Z0-9_]*\\d+/){4}\\d+)|(([a-zA-Z0-9_]*\\d+/){3}\\d+\\.\\d+)|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]*\\d+))|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]+))|([a-zA-Z0-9_-]*\\d+)|([a-zA-Z0-9_-]*\\d+\\.\\d+)|(mpls)|(dwdm)"
                ],
                "type": "string"
              }
            ]
          },
          "type-description": "An interface name specifying an interface type and\ninstance.\nInterface represents a string defining an interface\ntype and instance, e.g. MgmtEth0/4/CPU1/0 or\nTenGigE0/2/0/0.2 or Bundle-Ether9 or\nBundle-Ether9.98"
        },
        {
          "variable-name": "{$shutdown-var}",
          "type": "empty",
          "paths": {
            "path": [
              {
                "path": "/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration={%24act-var},{%24ifc-name-var}/shutdown",
                "path-description": "Shut the interface"
              }
            ]
          },
          "base-types": {
            "base-type": [
              {
                "type": "empty"
              }
            ]
          }
        }
      ]
    }
  }
}
```
