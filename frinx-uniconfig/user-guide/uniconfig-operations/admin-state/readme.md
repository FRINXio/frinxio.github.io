# Admin State

Admin state is used to lock, unlock or southbound-lock devices. Modification of
data on those devices is allowed or forbidden accordingly.

Three different states are currently supported:

* LOCKED - When a device is administratively locked, it is not possible 
  to modify its configuration and no changes are ever pushed to the device.
* UNLOCKED - The device is assumed to be operational. For all changes, an
  attempt is made to send them southbound. This is the default state for newly
  created devices.
* SOUTHBOUND_LOCKED - It is possible to configure the device, but no changes 
  are sent to the device. Useful when pre-provisioning devices.

The admin state is automatically added to the device during installation. You
can specify an admin state for a device as follows:
    
`"uniconfig-config:admin-state": "unlocked"`

An RPC is available for changing admin state after installation.

## RPC example

RPC input contains the device name and intended admin state.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:change-admin-state' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "node-id": "vnf20",
        "admin-state": "southbound_locked"
    }
}'
```

```RPC Response, Status: 204
```

## RPC example

`GET` request to get the actual state of the device.

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf/node=vnf20/uniconfig-config:admin-state' \
--header 'Authorization: Basic YWRtaW46e3twYXNzd29yZH19'
```

```json RPC Response, Status: 200
{
  "uniconfig-config:admin-state": "unlocked"
}
```

# RPC failed example

The device is in the `locked` admin state, and the user attempts to modify data
on the device.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node={{node_id}}/configuration/confdConfig/ssh/clientAliveCountMax' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "clientAliveCountMax": 4
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-message": "4af87c96-e49b-4e62-9217-316fe966d3d1: The commit RPC returned FAIL status. \n The node: '{{node_id}}' is currently in admin-state LOCKED.",
        "error-tag": "bad-element",
        "error-type": "rpc"
      }
    ]
  }
}
```