# UniConfig properties

UniConfig properties are application properties used to configure the application. They can be separated into three groups:

- **Runtime mutable properties** can be modified in runtime (using the *update-properties* RPC), their changes take effect in runtime and the properties are persisted in the database.
- **Database persisted properties** include all runtime mutable properties and some additional properties. These properties are stored in the database, which is always their primary source. With UniConfig Cloud Config, they remain constant across UniConfig instances in the same cluster and cannot be overridden via the *application properties* file.
- **Regular UniConfig properties** comprise all the remaining properties. These properties can always be changed using the *application.properties* file and can differ between UniConfig instances.

Database persisted properties can be changed or read in application runtime without restarting UniConfig by using UniConfig Cloud Config and the following RPCs:

- [RPC read-properties](../uniconfig-properties/rpc_read-properties)
- [RPC update-properties](../uniconfig-properties/rpc_update-properties)
