# Uniconfig properties

UniConfig can be extensively configured using application properties located in the *application.properties* file.

Application properties can be separated into three groups:
- **Runtime mutable properties** can be modified in runtime (using the *update-properties* RPC), their changes take effect in runtime and the properties are persisted in the database.
- **Database persisted properties** include all runtime mutable properties and some additional properties. These properties are stored in the database, which is always their primary source. With UniConfig Cloud Config, they remain constant across UniConfig instances in the same cluster and cannot be overridden via the *application properties* file.
- **Regular UniConfig properties** comprise all the remaining properties. These properties can always be changed using *application.properties* and can differ between UniConfig instances.

Database persisted properties include the following property prefixes:
- *crypto*
- *schema-settings*
- *callbacks*
- *notifications.kafka*
- *netconf-default-parameters*
- *gnmi-default-parameters*
- *cli-default-parameters*

These properties are stored in the *properties* table and are also known as **default properties**. They can be read and updated using the *read-properties RPC* and *update-properties RPC*.

!!!
After UniConfig is started, if default properties are found in the database, UniConfig will use the values in the database. For properties not found in the database, values from the first UniConfig instance after startup are used (by the *application.properties* file or env variables) and saved in the database for the next UniConfig instances.
!!!

## UniConfig Cloud Config

UniConfig Cloud Config is used to retain the same property values between distributed UniConfig instances connected via a message broker. It is largely the same technology as [Spring Cloud Config with JDBC backend](https://docs.spring.io/spring-cloud-config/reference/4.1/server/environment-repository/jdbc-backend.html) and [Spring Cloud Bus](https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/). The main difference is that UniConfig Cloud Config Server and Cloud Config Client are in the same project, while Spring requires a separate Cloud Config Server application.

By calling a special signal (the *Refresh Bus Endpoint* call) during runtime, the system sets the same value for persisted properties in all UniConfig instances. The signal is called immediately after mutable properties are modified using the *update-properties* RPC. The specific UniConfig instance calling the signal sends Kafka events containing the changed properties, while other instances read those properties from the database and use the refresh endpoint to update them in runtime.

### UniConfig startup with UniConfig Cloud Config

UniConfig startup with UniConfig Cloud Config:
![startup-with-ucc](startup-with-ucc.jpg)

* Before starting UniConfig, enable Cloud Config by using the following properties:

```
# Uniconfig cloud settings - depends on kafka connection
uniconfig.cloud.config.enabled=true
uniconfig.cloud.config.fail-fast=false
uniconfig.cloud.config.application-name=uniconfig
spring.cloud.bus.enabled=true
spring.kafka.bootstrap-servers=http://localhost:9092
management.endpoints.web.exposure.include=bus-refresh
# Disable cloud connection to Kafka for testing purpose, property spring.cloud.bus.enabled must be false
#spring.autoconfigure.exclude=org.springframework.cloud.stream.function.FunctionConfiguration
```

On startup, UniConfig checks the database for any default properties to configure:
- If default properties are found in the database, UniConfig manually refreshes its property values to use those in the database.
- If no default properties are found, UniConfig uses its existing properties and, once loaded, saves them in the database for the next UniConfig instances.

At the end of Spring initialisation, the *Refresh Bus Endpoint* is called. This refreshes default properties with the database values for all UniConfig instances connected via the Kafka refresh topic. A second refresh during the UniConfig startup cycle is required if several instances were started simultaneously and the database contains no property values to synchronize for properties (especially encryption keys).

At application runtime, if the *update-properties* RPC is used with default properties on input, UniConfig updates the properties in the database. It also calls the *Refresh Bus Endpoint*, which reloads properties for all UniConfig instances connected via Kafka.


### UniConfig startup without UniConfig Cloud Config

UniConfig startup without UniConfig Cloud Config:
![startup-without-ucc](startup-without-ucc.jpg)

* Before starting UniConfig, disable Cloud Config by using the following properties:

```
# Uniconfig cloud settings - depends on kafka connection
uniconfig.cloud.config.enabled=false
uniconfig.cloud.config.fail-fast=false
uniconfig.cloud.config.application-name=uniconfig
spring.cloud.bus.enabled=false
spring.kafka.bootstrap-servers=http://localhost:9092
management.endpoints.web.exposure.include=bus-refresh
# Disable cloud connection to Kafka for testing purpose, property spring.cloud.bus.enabled must be false
spring.autoconfigure.exclude=org.springframework.cloud.stream.function.FunctionConfiguration
```

On startup, UniConfig checks the database for any default properties to configure:
- If default properties are found in the database, UniConfig manually refreshes its property values to those in the database.
- If no default properties are found, UniConfig uses its existing properties and, once loaded, saves them in the database for the next UniConfig instances.

At the end of Spring initialisation, the *Refresh Bus Endpoint* is not called.

At application runtime, if the *update-properties* RPC is used with default properties on input, Uniconfig updates the properties in the database but not inside the application. This will therefore only affect the next UniConfig instance started after the properties are updated.
