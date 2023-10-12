# RPC update-properties

RPC updates property values and if *UniConfig Cloud Config* is enabled -> call *Refresh Bus Endpoint* which
update properties in runtime for all connected UniConfig instances. RPC updates only *default properties*, but not
*crypto* properties because those properties have different RPCs for updating 
(*change-encryption-status* RPC and *change-encryption-keys* RPC).

RPC sequence diagram with UniConfig Cloud Config enabled:

![update-with-ucc](update-with-ucc.jpg)

!!!
If *UniConfig Cloud Config* is disabled -> RPC update-properties will update the property value
only in the database, the application instance will still use the old property value, which may confuse the user.
In addition, after updating properties, if a new UniConfig instance is starting, that instance will use the updated
property value from the database, so UniConfig instances will use different values for the same property, as it shown
in the diagram below.

Recommendation: use this RPC with *UniConfig Cloud Config*. 
The exception is *callbacks.access-token* which is always up-to-date.
!!!

![update-without-ucc](update-without-ucc.jpg)

## RPC Examples

### Successful Example

RPC input contains the default properties with correct values.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:update-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
       "properties-map": [
            {
                "key": "netconf-default-parameters.session-timers.between-attempts-timeout",
                "value": "1500"
            },
            {
                "key": "gnmi-default-parameters.session-timers.internal-transaction-timeout",
                "value": "1000"
            },
            {
                "key": "notifications.kafka.transaction-notifications-enabled",
                "value": "false"
            }
       ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "update-properties-status": "There are 3 updated and 0 inserted properties successfully. Ignored keys: []"
  }
}
```

### Successful Example

RPC input contains the default property crypto.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:update-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
       "properties-map": [
            {
                "key": "crypto.encrypt-enabled",
                "value": "true"
            },
            {
                "key": "crypto.actual-encryption-public-key",
                "value": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
            }
       ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "update-properties-status": "There are 0 updated and 0 inserted properties successfully. Ignored keys: [crypto.encrypt-enabled, crypto.actual-encryption-public-key]"
  }
}
```

### Successful Example

RPC input contains incorrect property key.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:update-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
       "properties-map": [
            {
                "key": "aaa",
                "value": "1000"
            }
       ]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "update-properties-status": "There are 0 updated and 0 inserted properties successfully. Ignored keys: [aaa]"
  }
}
```

### Failed example

RPC input contains the default properties with incorrect values.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:update-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
       "properties-map": [
            {
                "key": "netconf-default-parameters.session-timers.between-attempts-timeout",
                "value": "1500"
            },
            {
                "key": "gnmi-default-parameters.session-timers.internal-transaction-timeout",
                "value": "true"
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
        "error-tag": "invalid-value",
        "error-info": "For input string: \"true\"",
        "error-message": "Integer property has invalid value true.",
        "error-type": "rpc"
      }
    ]
  }
}
```

### Failed example

RPC input contains the default properties with incorrect values.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:update-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
       "properties-map": [
            {
                "key": "notifications.kafka.transaction-notifications-enabled",
                "value": "1000"
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
        "error-tag": "invalid-value",
        "error-info": "The String did not match either specified value",
        "error-message": "Boolean property has invalid value 1000.",
        "error-type": "rpc"
      }
    ]
  }
}
```
