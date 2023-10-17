# UniConfig Properties

UniConfig properties are application properties that can be used to configure the application.
These parameters are divided into 3 groups:
* runtime mutable properties - can be modified in runtime (via update-properties RPC)
  and the change takes effect in runtime, and these properties are persisted in database.
* database persisted properties - includes all runtime mutable properties
  with some additional properties. These properties are stored in database and database is always the primary source
  for these properties. Meaning that with *UniConfig Cloud Config* they stay the same across UniConfig instances
  in the same cluster and cannot be overridden via *application.properties*.
* regular UniConfig properties - this is the rest of them, can be always changed via *application.properties* and
  can be different per UniConfig instance.

Database persisted properties can be changed or read in application runtime without restarting UniConfig 
using *UniConfig Cloud Config* and these RPCs:


## Read properties

[!ref text="Read properties"](../uniconfig-properties/rpc_read-properties)

## Update properties

[!ref text="Update properties"](../uniconfig-properties/rpc_update-properties)
