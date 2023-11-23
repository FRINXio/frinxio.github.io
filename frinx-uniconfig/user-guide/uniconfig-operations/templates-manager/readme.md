# Templates Manager

## Overview

Templates can be utilised to reuse some configuration and more easily
apply this configuration into target UniConfig nodes.

Basic properties of templates in UniConfig:

- All templates are stored under the `templates` topology and each
    template is represented by a separate `node` list entry.
- The entire template configuration is placed under the
    `frinx-uniconfig-topology:configuration` container in the
    *Configuration* datastore. Because of this, the configuration of a template
    can be accessed and modified in the same way as a UniConfig node.
- Templates are validated against a single schema context. The schema
    context is selected when a template is created using
    the `uniconfig-schema-repository` query
    parameter. The value of the query parameter defines the name of the schema
    repository that is placed under the UniConfig distribution in the form of
    the directory.

Currently implemented template features:

- **Variables** - Used for parametrisation of templates.
- **Tags** - Can be used to select an operation that is applied
    to the specific subtree when the template is applied to a UniConfig node.

!!!
Schema validation of leaves and leaf-lists is adjusted so that it can
accept both string with variables and original YANG type.
!!!

- [RPC apply-template](../templates-manager/rpc_apply-template)
- [RPC create-multiple-templates](../templates-manager/rpc_create-multiple-templates)
- [RPC get-template-info](../templates-manager/rpc_get-template-info)
- [RPC get-template-nodes](../templates-manager/rpc_get-template-nodes)
- [RPC upgrade-template](../templates-manager/rpc_upgrade-template)

## Latest-schema

Latest-schema defines the name of the schema repository whose built schema context is used for template validation.

Latest-schema is used only if the `uniconfig-schema-repository` query parameter is not defined when creating the template.
If the query parameter is defined, latest-schema is ignored.

### Configuring latest-schema

Latest-schema can be set using a PUT request. It is placed in the *Config* datastore. The name of the directory must point
to an existing schema repository placed under the UniConfig distribution.

```bash Set the latest schema
curl --location --request PUT 'http://localhost:8181/rests/data/schema-settings:schema-settings/latest-schema' \
--header 'Content-type: application/json' \
--data-raw '{
    {
        "latest-schema" : "schemas-1"
    }
}'
```

```PUT response, Status: 201
```

GET request can be used to check if latest-schema is placed in the *config* datastore.

```bash Read the latest schema
curl --location --request GET 'http://localhost:8181/rests/data/schema-settings:schema-settings/latest-schema' \
--header 'Accept: application/json'
```

```json GET response, Status: 200
{
  "latest-schema" : "schemas-1"
}
```

### Auto-upgrading latest-schema

Latest-schema can be automatically upgraded by UniConfig after installing a new YANG repository. YANG repository
is installed after deploying of new type of NETCONF/GRPC device or after manual invocation of RPC for loading
of new YANG repository from directory.

In order to enable auto-upgrading process, `latestSchemaReferenceModuleName` must be specified in the
*application.properties* file:

```properties UniConfig templates configuration (config/application.properties)
# Template settings
templates.enabled=false
templates.latest-schema-reference-module-name=system
templates.enabled-templates-upgrading=false
templates.backup-templates-limit=7
```

After new YANG repository is installed, UniConfig will look for revision of the 
`latestSchemaReferenceModuleName` module in the repository. If the revision found is more recent than the last cached
revision, UniConfig will automatically write identifier of the fresh repository into 'latest-schema' configuration.
Afterward, 'latest-schema' is used by UniConfig the same way as it would be written manually via RESTCONF.

## Variables

Using variables it is possible to parametrise values in the template.
Structural parametrisation is not currently supported.

Properties:

- Format of the variable: '{\$variable-id}'.
- Variables can be set to each leaf and leaf-list in the template.
- Single leaf or leaf-list may contain multiple variables.
- Key of the list can also contain variable.
- Variables are substituted by provided values at the application of
    template to UniConfig node.
- It is possible to escape characters of the variable pattern ('\$',
    '{', '}'), so they will be interpreted as value and not part of the
    variable.
- Variable identifier may contain any UTF-8 characters. Characters
    '\$', '{', '}' must be escaped, if they are part of the variable
    identifier.

### Examples with variables

**A. Leaf with one variable**

- The following example shows 2 leaves with 2 variables: 'var-1' and
    'var-2'.

```
{
  "leaf-a": "{$var-1}",
  "leaf-b": "{$var-2}"
}
```

Application of values '10' and 'false' to 'var-1', and 'var-2'. Leaf
'leaf-a' has 'int32' type and 'leaf-b' has 'boolean' type.

```
{
  "leaf-a": 10,
  "leaf-b": false
}
```

**B. Leaf with multiple variables**

- Leaf 'leaf-a' contains 2 variables and surrounding text that is not
    part of any variable.
- Leaf 'leaf-b' contains 2 variable without additional text -
    substituted values of these variables are concatenated at
    application of template.

```
{
  "leaf-a": "custom {$var-x} text {$var-y}",
  "leaf-b": "{$var-1}{$var-2}"
}
```

Application of values - 'var-x': 'next', 'var-y': '7', 'var-1': '10',
'var-2': '9'. Leaf 'leaf-a' has 'string' type and 'leaf-b' has 'int32'
type.

```
{
  "leaf-a": "custom next text 7",
  "leaf-b": 109
}
```

**C. Leaf-list with one variable**

- Leaf-list 'leaf-list-a' contains variable with identifier 'var-x'.
- This variable can be substituted by one or multiple values. If
    multiple values are provided in the apply-template RPC, they are
    'unwrapped' to the leaf-list in form of next leaf-list entries.

```
{
  "leaf-list-a": [
    "{$var-x}"
  ]
}
```

Substitution of 'var-x' with numbers '10', '20', '30' results in
('int32' type):

```
{
  "leaf-list-a": [
    10, 20, 30
  ]
}
```

**D. Leaf-list with multiple variables**

- Leaf-list 'leaf-list-a' contains 2 variables with identifiers
    'var-a' and 'var-2'. String "str3" represents constant value.
- It is possible to substitute both variables with one or multiple
    variables.

```
{
  "leaf-list-a": [
    "{$var-a}",
    "str3",
    "{$var-b}"
  ]
}
```

Substitution of 'var-a' with texts 'str1', 'str2' and 'var-b' with
'str4' results in ('string' type):

```
{
  "leaf-list-a": [
    "str1",
    "str2",
    "str3",
    "str4"
  ]
}
```

!!!
If leaf-list is marked as "ordered-by user", then the order of
leaf-list elements is preserved during substitution process.
!!!

**E. Leaf-list with entry that contains multiple variables**

- Leaf-list 'leaf-list-a' contains 2 variables inside one leaf-list
    entry: 'var-a' and 'var-b'.
- Both variables must be substituted by the same number of values.

```
{
  "leaf-list-a": [
    "ratio: {$var-a}%{$var-b}",
    "strX"
  ]
}
```

- Application of following values to variables 'var-a' and 'var-b':
    'var-a' = ['10', '20', '30'], 'var-b' = ['50', '70', '60'].

```
{
  "leaf-list-a": [
    "ratio: 10%50",
    "ratio: 20%70",
    "ratio: 30%60",
    "strX"
  ]
}
```

**F. Leaves and leaf-lists with escaped special characters**

- The following example demonstrates escaping of special characters
    outside the variable identifier (leaf-list 'leaf-list-a') and
    inside the variable identifier (leaf 'leaf-a').
- Unescaped identifier of the leaf 'leaf-a': 'var-{2}'.

```
{
  "leaf-list-a": [
    "{$var-1} text \\{\\$this-is-not-variable\\}"
  ],
  "leaf-a": "{$var-\\{2\\}}"
}
```

Substitution of 'var-1' by 'prefix' and 'var-{2}' by '10':

```
{
  "leaf-list-a": [
    "prefix text {$this-is-not-variable}"
  ],
  "leaf-a": 10
}
```

## Tags

By default, all templates have assigned 'merge' tag to the root
'configuration' container - if template doesn't explicitly define next
tags in the data-tree, then the whole template is merged to target
UniConfig node configuration at execution of apply-template RPC.
However, it is possible to set custom tags to data-tree elements of the
template.

Properties:

- Tags are represented in UniConfig using node attributes with the
    following identifier: 'template-tags:operation'.
- In RESTCONF, attributes are encoded using special notation that is
    explained in the 'RESTCONF' user guide.
- Tags are inherited through the data-tree of the template. If
    data-tree element doesn't define any tag, then it is inherited from
    parent element.
- Only single tag can be applied to one data node.
- Tags can be applied to following YANG structures: container, list,
    leaf-list, leaf, list entry, leaf-list entry.

Currently, the following tags are supported:

- **merge**: Merges with a node if it exists, otherwise creates the
    node.
- **replace**: Replaces a node if it exists, otherwise creates the
    node.
- **delete**: Deletes the node.
- **create**: Creates a node. The node can not already exist. An error
    is raised if the node exists.
- **update**: Merges with a node if it exists. If it does not exist,
    it will not be created.

### Examples with tags

**A. Tags applied to container, list, and leaf**

Template with name 'user\_template' that contains 'merge', 'replace',
and 'create' tags:

```
{
  "node": [
    {
      "node-id": "user_template",
      "frinx-uniconfig-topology:configuration": {
        "system:system": {
          "@": {
            "template-tags:operation": "update"
          },
          "users": {
            "@": {
              "template-tags:operation": "replace"
            },
            "#": [
              {
                "name": "test-${username}",
                "login": "{$login}",
                "role": "{$role}",
                "password": {
                  "@": {
                    "template-tags:operation": "create"
                  },
                  "#": "{$password}"
                }
              }
            ]
          }
        }
      }
    }
  ]
}
```

Description of all operations in the correct order that are done based
on the defined tags:

- Container 'configuration' will be merged to target UniConfig node
    (implicit root operation).
- Container 'system:system' will be updated - its content is merged
    only, if it has already been created.
- The whole list 'users' will be replaced in the target UniConfig node.
- Leaf named 'password' will be created at the target UniConfig node -
    it cannot exist under 'users' list entry, otherwise the error will
    be raised.

**B: Tags applied to leaf-list, leaf-list entry, and list entry**:

The following JSON represents content of sample template with multiple
tags:

- 'replace' tag is applied to single list 'my-list' entry
- 'merge' tag is applied to whole 'leaf-list-a' leaf-list
- 'create' tag is applied to whole 'leaf-list-b' leaf-list
- 'delete' tag is applied to single leaf-list 'leaf-list-b' entry with
    value '10'

```
{
  "c1": {
    "my-list": [
      {
        "key": "k1",
        "value": 10,
        "@": {
          "template-tags:operation": "replace"
        }
      }
    ],
    "leaf-list-a": {
      "@": {
        "template-tags:operation": "merge"
      },
      "#": [
        10,
        20
      ]
    },
    "leaf-list-b": {
      "@": {
        "template-tags:operation": "create"
      },
      "#": [
        {
          "@": {
            "template-tags:operation": "delete"
          },
          "#": 10
        }
      ]
    }
  }
}
```

## Creating a template

A new template can be created by sending PUT request to new template
node under 'templates' topology with populated 'configuration'
container. Name of the template equals to name of the 'node' list entry.
This RESTCONF call must contain specified schema cache repository using
the 'uniconfig-schema-repository' query parameter in order to
successfully match sent data-tree with correct schema context (it is
usually associated with some type of NETCONF device).

### Example

The following example shows creation of new template with name
'interface\_template' using 'schemas\_1' schema repository. The body of
the PUT request contains whole 'configuration' container.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration?uniconfig-schema-repository=schemas_1' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "frinx-uniconfig-topology:configuration": {
        "interfaces:interfaces": [
            {
                "name": "eth-0/{$interface-id}",
                "enabled": "{$enabled}",
                "description": "purpose: '\''{$description}'\'''\''",
                "type": "{$type}",
                "unit": [
                    {
                        "@": {
                            "template-tags:operation": "replace"
                        },
                        "vlan-id": "{$unit-id}",
                        "enable": false,
                        "name": "{$unit-id}",
                        "family-inet": {
                            "address": [
                                {
                                    "@": {
                                        "template-tags:operation": "update"
                                    },
                                    "address": "{$ip-address}"
                                }
                            ]
                        }
                    }
                ]
            }
        ]
    }
}'
```

```RPC Response, Status: 204
```

## Read/update/delete template

All CRUD operations with templates can be done using standard RESTCONF
PUT/DELETE/POST/PLAIN PATCH methods. As long as template contains some
data under 'configuration' container, next RESTCONF calls, that work
with templates, don't have to contain 'uniconfig-schema-repository'
query parameter, since type of the device is already known.

### Examples - RESTCONF operations

Reading specific subtree under 'interface\_template' - unit with name
'{\$unit-id}' that is placed under interface with name
'eth-0/{\$interface-id}'.

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/interfaces:interfaces=eth-0%2F%7B%24interface-id%7D/unit=%7B%24unit-id%7D?content=config' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
  "unit": [
    {
      "@": {
        "template-tags:operation": "replace"
      },
      "vlan-id": "{$unit-id}",
      "enable": false,
      "name": "{$unit-id}",
      "family-inet": {
        "address": [
          {
            "@": {
              "template-tags:operation": "update"
            },
            "address": "{$ip-address}"
          }
        ]
      }
    }
  ]
}
```

Changing 'update' tag of the 'address' list entry to 'create' tag using
PLAIN-PATCH RESTCONF method.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=templates/node=interface_template/frinx-uniconfig-topology:configuration/interfaces:interfaces=eth-0%2F%7B%24interface-id%7D/unit=%7B%24unit-id%7D/family-inet/address=%7B%24ip-address%7D' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "address": [
        {
            "@": {
                "template-tags:operation": "create"
            },
            "address": "{$ip-address}"
        }
    ]
}'
```

```json RPC Response, Status: 204
```
