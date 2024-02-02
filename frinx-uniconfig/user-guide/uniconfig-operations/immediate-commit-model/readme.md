# Immediate Commit Model

The immediate commit creates new transactions for every call of an RPC.
The transaction is then closed so that no lingering data remains.

The sequential diagram below illustrates how the process works for reading data
(GET request).

![Get Request](get-request.png)

Similarly, the following diagram illustrates the process for putting data (PUT
request).

![Put Request](put-request.png)

The key difference in the two diagrams is that editing data (PUT, PATCH, DELETE,
POST) and RPC calls in the database must be committed, hence there is an
additional call of the **commit RPC**. This commit ensures that the transaction
is closed. For reading data, it is necessary to close the transaction in a
different manner, because no data was changed and calling a commit is not
necessary.

When calling the **sync-from-network RPC**, it internally calls the
**replace-config-with-operational RPC**. Note that this only works when
using the Immediate Commit Model.

## Configuration

Configurations related to UniConfig transactions are placed in the
**config/application.properties** file. You can also turn off the Immediate
Commit Model and use the 
[Build and Commit Model](../build-and-commit-model/readme.md) instead.


```properties Transaction property snippet
# Boolean value to enable or disable the Immediate Commit Model. The default value is 'true'.
# If disabled, only manually created transactions can exist.
transactions.immediate-commit-enabled=true
```

## RPC examples

### Successful example

RPC input contains a new interface that is added to the existing ones.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=Loopback123' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
"interface":
    [
        {
            "name": "Loopback123",
            "config": {
                "type": "iana-if-type:softwareLoopback",
                "name": "Loopback123",
                "description": "testing",
                "enabled": true
            }
        }
    ]
}'
```

```json RPC Response, Status: 200
{
}
```

After putting the data into the database, they are automatically committed and
can be viewed.

```bash RPC Request
curl --location --request GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=Loopback123?content=nonconfig' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
"frinx-openconfig-interfaces:interface": [
        {
            "name": "Loopback123",
            "config": {
                "type": "iana-if-type:softwareLoopback",
                "enabled": true,
                "description": "testing",
                "name": "Loopback123"
            }
        }
    ]
}
```

### Failed example

RPC input contains a value that is not supported.

```bash RPC Request
curl --location --request PUT 'http://localhost:8181/rests/data/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=Loopback123' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
"interface":
    [
        {
            "name": "Loopback123",
            "config": {
                "type": "iana-if-type:softwareLoopback",
                "name": "Loopback123",
                "mtu": 1400,
                "description": "testing",
                "enabled": true
            }
        }
    ]
}'
```

```json RPC Response, Status: 500
{
   "errors": {
      "error": [
         {
            "error-message": "UniconfigTransaction[transactionId=f5184f5d-c0bc-4abc-a591-eeddb704eac1, creationTime=2021-10-12T12:06:00.028925Z, readWriteTx=f5184f5d-c0bc-4abc-a591-eeddb704eac1]: The commit RPC returned FAIL status. \\n Bulk update failed because: Tue Oct 12 12:06:09.478 UTC\\r\\n!! SEMANTIC ERRORS: This configuration was rejected by \\r\\n!! the system due to semantic errors. The individual \\r\\n!! errors with each failed configuration command can be \\r\\n!! found below.\\r\\n\\r\\n\\r\\ninterface Loopback123\\r\\n mtu 1400\\r\\n!!% This operation is not supported: The interface owner has not registered support for MTU\\r\\n!\\r\\nend",
            "error-tag": "operation-failed",
            "error-type": "rpc"
         }
      ]
   }
}
```
