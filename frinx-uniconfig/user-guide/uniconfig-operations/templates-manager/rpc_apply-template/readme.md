# Application of template

Template can be applied to UniConfig nodes using 'apply-template' RPC.
This procedure does following steps:

1. **Read template** - Reading of template configuration from
   'templates' topology in Configuration datastore.
2. **String-substitution** - Substitution of variables by provided
   values or default values, if there aren't any provided values for
   some variables and leaf/leaf-list defines a default values. If some
   variables cannot be substituted (for example, user forgot to specify
   input value of variable), an error will be returned.
3. **Version-drop** - Conversion of template into target schema context
   that is used by target UniConfig node. This component also drops
   unsupported data from input template. Because of this feature, it is
   possible to apply template between different versions of devices
   with different revisions of YANG schemas but with similar structure.
   Version-drop is also aware of 'ignoredDataOnWriteByExtensions'
   RESTCONF filtering mechanism.
4. **Application of tags** - Data-tree of the template is streamed and
   data is applied to target UniConfig node based on set tags on data
   elements, recursively. UniConfig node configuration is updated only
   in the Configuration datastore.

Description of input RPC fields:

- **template-node-id**: Name of the existing input template.
- **uniconfig-node**: List of target UniConfig nodes to which template
  is applied ('uniconfig-node-id' is the key).
- **uniconfig-node-id**: Target UniConfig node identifier.
- **variable**: List of variables and substituted values that must be
  used during application of template to UniConfig node. Variables
  must be set per target UniConfig node since it is common, that
  values of variables should be different on different devices. Leaf
  'variable-id' represents the key of this list.
- **variable-id**: Unescaped variable identifier.
- **leaf-value**: Scalar value of the variable. Special characters
  ('\$', '{', '}') must be escaped.
- **leaf-list-values**: List of values - it can be used only with
  leaf-lists. Special characters ('\$', '{', '}') must be escaped.

Description of output RPC fields:

- Successful response: returns http status code 200 without fields.

- Failed response: returns http status code 400-500 and error with
  fields shown below:
- **error-type** : Type of the error.
- **error-tag** : Tag of the error, also determining http status code.
- **error-message** : Description of the error that occurred during
  application of template.
- **error-info** : Additional information related to error. For example,
  node identification, topology identification.

The following sequence diagram and nested activity diagram show process
of 'apply-template' RPC in detail.

![RPC apply-template](rpc_apply-template/apply_template.svg)

![Processing template configuration](processing_template.svg)

## RPC Examples

### Successful Example

Successful application of the template 'service\_group' to 2 UniConfig
nodes - 'R1' and 'R2'.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/operations/template-manager:apply-template' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-node-id": "SERVICE_GROUP",
        "uniconfig-node": [
            {
                "uniconfig-node-id": "R1",
                "variable": [
                    {
                        "variable-id": "svc-id",
                        "leaf-value": 1
                    },
                    {
                        "variable-id": "svc-1",
                        "leaf-value": "my-sng"
                    },
                    {
                        "variable-id": "svc-type",
                        "leaf-value": "other"
                    },
                    {
                        "variable-id": "svc-service",
                        "leaf-list-values": [
                            "sdwan",
                            "cgnat"
                        ]
                    }
                ]
            },
            {
                "uniconfig-node-id": "R2",
                "variable": [
                    {
                        "variable-id": "svc-id",
                        "leaf-value": 1
                    },
                    {
                        "variable-id": "svc-1",
                        "leaf-value": "custom-sng"
                    },
                    {
                        "variable-id": "svc-type",
                        "leaf-value": "internal"
                    },
                    {
                        "variable-id": "svc-service",
                        "leaf-list-values": [
                            "sdwan"
                        ]
                    }
                ]
            }
        ]
    }
}'
```

```RPC Response, Status: 200
```

### Failed Example

Failed application of the template 'TEMP1' - template doesn't exist.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/operations/template-manager:apply-template' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-node-id": "TEMP1",
        "uniconfig-node": [
            {
                "uniconfig-node-id": "R1",
                "variable": [
                    {
                        "variable-id": "remote-ip",
                        "leaf-value": "172.30.15.1"
                    },
                    {
                        "variable-id": "mode",
                        "leaf-value": "vpn"
                    }
                ]
            }
        ]
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "data-missing",
        "error-message": "Template with node ID 'TEMP1' is not present in CONFIG datastore.",
        "error-info": {
          "node-id": "R1"
        }
      }
    ]
  }
}
```

### Failed Example

Failed application of the template 'service\_group' to 2 UniConfig nodes
- 'R1' and 'R2' - user hasn't provided values for all required
  variables.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/operations/template-manager:apply-template' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-node-id": "REDUNDANCY_TEMPLATE",
        "uniconfig-node": [
            {
                "uniconfig-node-id": "R1",
                "variable": [
                    {
                        "variable-id": "local-ip",
                        "leaf-value": "172.30.15.1"
                    },
                    {
                        "variable-id": "redundant-mode",
                        "leaf-value": "service"
                    }
                ]
            },
            {
                "uniconfig-node-id": "R2",
                "variable": [
                    {
                        "variable-id": "local-ip",
                        "leaf-value": "172.30.15.1"
                    },
                    {
                        "variable-id": "redundant-mode",
                        "leaf-value": "service"
                    },
                    {
                        "variable-id": "min-interval",
                        "leaf-value": 100
                    }
                ]
            }
        ]
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
        "error-message": "String substitution failed: Node /network-topology:network-topology/topology=templates/node=redundancy_template/frinx-uniconfig-topology:configuration/ha:redundancy/intra-chassis/bfd-liveness-detection/transmit-interval/minimum-interval has defined variable/s: '[min-interval]', but there is not provided or default value for all of these variables",
        "error-info": {
          "node-id": "R2"
        }
      }
    ]
  }
}
```

### Failed Example

Failed application of the template 'redundancy\_template' to UniConfig
node 'dev1' - type of the substituted variable value is invalid (failed
regex constraint).

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/operations/template-manager:apply-template' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-node-id": "redundancy_template",
        "uniconfig-node": [
            {
                "uniconfig-node-id": "dev1",
                "variable": [
                    {
                        "variable-id": "local-ip",
                        "leaf-value": "172.30.15.1s"
                    },
                    {
                        "variable-id": "redundant-mode",
                        "leaf-value": "service"
                    },
                    {
                        "variable-id": "control-mode",
                        "leaf-value": "vcn1"
                    },
                    {
                        "variable-id": "min-interval",
                        "leaf-value": "50"
                    }
                ]
            }
        ]
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
        "error-message": "Value '172.30.15.1s' cannot be applied to leaf /network-topology:network-topology/topology=templates/node=redundancy_template/frinx-uniconfig-topology:configuration/ha:redundancy/inter-chassis/local-ip - it accepts only values with following YANG types: [type: string, constraints: [Length[[0..2147483647]], PatternConstraintImpl{regex=^(?:(([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])(%[\\p{N}\\p{L}]+)?)$, errorAppTag=invalid-regular-expression}], type: string, constraints: [Length[[0..2147483647]], PatternConstraintImpl{regex=^(?:((:|[0-9a-fA-F]{0,4}):)([0-9a-fA-F]{0,4}:){0,5}((([0-9a-fA-F]{0,4}:)?(:|[0-9a-fA-F]{0,4}))|(((25[0-5]|2[0-4][0-9]|[01]?[0-9]?[0-9])\\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9]?[0-9])))(%[\\p{N}\\p{L}]+)?)$, errorAppTag=invalid-regular-expression}, PatternConstraintImpl{regex=^(?:(([^:]+:){6}(([^:]+:[^:]+)|(.*\\..*)))|((([^:]+:)*[^:]+)?::(([^:]+:)*[^:]+)?)(%.+)?)$, errorAppTag=invalid-regular-expression}]]",
        "error-info": {
          "node-id": "dev1"
        }
      }
    ]
  }
}
```
