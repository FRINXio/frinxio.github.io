# RPC create-multiple-templates

This RPC can be used to create one or more new templates.

Templates are parsed and written in parallel for better performance. If
specified templates already exist, their configuration is replaced. Execution of
the RPC is atomic, meaning that either all templates are successfully created or no
changes are made to the UniConfig transaction.

RPC input fields:

- `template-name` -  Name of the created template.
- `yang-repository` - YANG schema repository used to parse template
  configuration. The default value is `latest`.
- `template-configuration` - The entire template configuration.
- `tags` - List of template tags written on the specified paths in all created
  templates. A specified tag type must be prefixed with the `template-tags`
  module name based on RFC-8040 formatting of identityref.

RPC output fields:

- Successful response: Returns HTTP status code 200 with no fields.

- Failed response: Returns HTTP status code 400&ndash;500 and an error with
  the following fields:
  - `error-type` - Error type.
  - `error-tag` - Error tag, also determines HTTP status code.
  - `error-message` - Description of the error that occurred when the template
     was applied.
  - `error-info` - Additional information related to the error. For example,
     node identification, topology identification.

Mandatory fields are `template-name` and `template-configuration`.

## RPC examples

### Successful example

Successfully creating templates.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/template-manager:create-multiple-templates' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "templates": [
            {
                "template-name": "Template-name1",
                "yang-repository": "schema-1779846763",
                "template-configuration": {
                    "system:system-config": {
                        "@": {
                            "template-tags:operation": "replace"
                        },
                        "timezone": "CEST"
                    }
                }
            },
            {
                "template-name": "Template-name2",
                "yang-repository": "schema-1779846763",
                "template-configuration": {
                    "interfaces:interfaces": [
                        {
                            "name": "e0",
                            "description": "test"
                        }
                    ]
                }
            },
            {
                "template-name": "Template-name3",
                "template-configuration": {
                    "dhcp:dhcp": {
                        "enabled": true
                    },
                    "services": ["dhcp"]
                }
            }
        ]
    }
}'
```

```RPC Response, Status: 204
```

### Successful example

Creating two templates with separately specified template tags:
- `replace` tag is added to `/acl/category` and `/services/group=default/types`
  elements
- `create` tag is added to `/services` element

```bash RPC request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/template-manager:create-multiple-templates' \
--header 'Content-Type: application/json' \
--data-raw '{
  "input": {
    "templates": [
      {
        "template-name": "Template-name1",
        "yang-repository": "schema1",
        "template-configuration": {
          "acl": {
            "rules": [
              {
                "name": "test-name",
                "category": [
                  "*"
                ]
              }
            ]
          }
        }
      },
      {
        "template-name": "Template-name2",
        "template-configuration": {
          "services": {
            "group": [
              {
                "name": "default",
                "id": 0,
                "type": "internal",
                "types": [
                  "test"
                ]
              }
            ]
          }
        }
      }
    ],
    "tags": [
      {
        "tag": "template-tags:replace",
        "paths": [
          "/acl/category",
          "/services/group=default/types"
        ]
      },
      {
        "tag": "template-tags:create",
        "paths": [
          "/services"
        ]
      }
    ]
  }
}
```

```RPC response, Status: 204
```

### Failed example

Cannot find the specified YANG schema repository.

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/template-manager:create-multiple-templates' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "templates": [
            {
                "template-name": "Template-name1",
                "yang-repository": "schema-1",
                "template-configuration": {
                    "system:system-config": {
                        "@": {
                            "template-tags:operation": "replace"
                        },
                        "timezone": "CEST"
                    }
                }
            },
            {
                "template-name": "Template-name2",
                "yang-repository": "schema-2",
                "template-configuration": {
                    "system:system-config": {
                        "@": {
                            "template-tags:operation": "replace"
                        },
                        "timezone": "CEST"
                    }
                }
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
        "error-type": "uniconfig-error",
        "error-tag": "missing-element",
        "error-message": "Failed to find schema repository: schema-1",
        "error-info": {
          "node-id": "Template-name1"
        }
      }
    ]
  }
}
```

### Failed example

Failed to parse template configuration.

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/template-manager:create-multiple-templates' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "templates": [
            {
                "template-name": "Template-name1",
                "yang-repository": "schema-1",
                "template-configuration": {
                    "system:system-config": {
                        "@": {
                            "template-tags:operation": "replace"
                        },
                        "timezone": "CEST"
                    }
                }
            },
            {
                "template-name": "Template-name2",
                "yang-repository": "schema-2",
                "template-configuration": {
                    "dhcp:dhcp": {
                        "enabled": true
                    },
                    "service": ["dhcp"]
                }
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "invalid-value",
        "error-message": "Node with name 'service' has not been found under 'configuration'",
        "error-info": {
          "node-id": "Template-name2"
        }
      }
    ]
  }
}
```
