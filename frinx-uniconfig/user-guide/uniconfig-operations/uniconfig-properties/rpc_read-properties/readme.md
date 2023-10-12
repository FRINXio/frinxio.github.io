# RPC read-properties

RPC reads *default properties* from database if property keys exist. If any property key
doesn't exist in database, then this key will be returned to user in section 'ignored keys'.
RPC works the same both with UniConfig Cloud Config enabled and disabled.

![read](read.jpg)

!!!
If *UniConfig Cloud Config* is disabled, then RPC read-properties will read the property value
from the database, which may be different from the application instance property value.
!!!

## RPC Examples

### Successful Example

RPC input contains *default property* keys.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:read-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "property-keys": [
          "notifications.kafka.audit-logs-enabled",
          "notifications.kafka.netconf-notifications-enabled", 
          "notifications.kafka.gnmi-notifications-enabled"
        ]
    }
}'
```

```json RPC Response, Status: 200
{
    "output": {
        "read-properties-status": "There are 3 from 3 properties read successfully. Ignored keys: []",
        "properties-map": [
            {
                "key": "notifications.kafka.audit-logs-enabled",
                "value": "true"
            },
            {
                "key": "notifications.kafka.netconf-notifications-enabled",
                "value": "true"
            },
            {
                "key": "notifications.kafka.gnmi-notifications-enabled",
                "value": "true"
            }
        ]
    }
}
```

### Successful Example

RPC input contains properties that are not default or are private (crypto keys and crypto types).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:read-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "property-keys": ["crypto.actual-encryption-cipher-type", "crypto.encrypt-enabled"]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "read-properties-status": "There are 1 from 2 properties read successfully. Ignored keys: [crypto.actual-encryption-cipher-type]",
    "properties-map": [
      {
        "key": "crypto.encrypt-enabled",
        "value": "true"
      }
    ]
  }
}
```

### Successful Example

The RPC input consists of properties that are not in the database.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:read-properties' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "property-keys": ["bbb", "aaa"]
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "read-properties-status": "There are 0 from 2 properties read successfully. Ignored keys: [bbb, aaa]"
  }
}
```
