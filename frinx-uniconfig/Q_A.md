---
icon: question
label: FAQ
---

# FAQ

## What is the datastore used in FRINX UniConfig ?

UniConfig uses a custom in-memory database that is part of MD-SAL and is a very fast storage for YANG modeled data.

The datastore is used only for caching data in the scope of a single transaction. For persistence purposes, UniConfig uses a PostgreSQL database.

---

## Are service instances stored in the UniConfig layer of FRINX ?

Only the "outputs" of a service are stored and managed by UniConfig (for example, a service that generates BGP config for 10 devices, which is pushed into UniConfig).

Services are responsible for managing their own configuration/operational state, and rely on the same database to store configuration or operational data.

---

## How does FRINX deal with model changes ?

OpenConfig models are compiled as part of UniConfig and can therefore be changed only before compilation.

On the other hand, NETCONF models can be dynamically loaded from a device and can also be manually updated using a [dedicated RPC](https://docs.frinx.io/frinx-uniconfig/user-guide/network-management-protocols/uniconfig_netconf/#registration-or-refreshing-of-netconf-cache-repository-using-rpc).

---

## Does FRINX provide auto rollback on all affected devices if a transaction fails on one or more devices ?

Yes, all onboarded devices have full rollback implemented.

You can also disable auto-rollback in UniConfig, so that successfully configured devices will keep their configuration. This is done by setting the `do-rollback` flag to `False` in the input for the **commit RPC**.

---

## Is it possible to display differences between the actual device configuration and the operational datastore while synchronizing the configuration into FRINX ?

Yes, follow these steps:

1. Sync (update `operational`).
2. Show diff.
3. Drop the changes from the device by replacing `operational` with `config`.

---

## Is any NETCONF device fully supported, or must OpenConfig be mapped to NETCONF as well ?

You can either use the native device models (via UniConfig native) or existing translation units between OpenConfig and vendor models.

---

## Are the libraries used to access the Config Data Store, model driven ?

UniConfig has a DataBroker interface and the concept of **InstanceIdentifier**. These are the model driven APIs for data access.

For more information, see [MD-SAL basic concepts](https://docs.opendaylight.org/projects/mdsal/en/latest/#basic-concepts).

---

## What would an access to the configuration data store look like in code ?

**A.** To demonstrate the API, this example reads **InterfaceConfigurations** from CONF DS and puts it back to CONF DS.

```
ReadWriteTransaction rwTx = dataBroker.newReadWriteTransaction();
InstanceIdentifier`<InterfaceConfigurations>` iid = InstanceIdentifier.create(InterfaceConfigurations.class);
InterfaceConfigurations ifcConfig = xrNodeReadTx.read(LogicalDatastoreType.CONFIGURATION, iid).checkedGet();
rwTx.put(LogicalDatastoreType.CONFIGURATION, iid, ifcConfig);
rwTx.submit();
```
**B.** This example reads **InterfaceConfigurations** from OPER DS.

```
ReadWriteTransaction rwTx = dataBroker.newReadWriteTransaction();
InstanceIdentifier`<InterfaceConfigurations>` iid = InstanceIdentifier.create(InterfaceConfigurations.class);
InterfaceConfigurations ifcConfig = xrNodeReadTx.read(LogicalDatastoreType.OPERATIONAL, iid).checkedGet();
```

---

## Is it possible in FRINX to run a transaction on two disjunct sets of devices simultaneously ?

UniConfig supports the build-and-commit model, which makes it possible to configure devices in isolated transactions and commit them in parallel. If there are conflicts between configured sets of devices, the second transaction that is committed will fail (however, this cannot happen on disjunct sets of devices).

---

## What access control measures does FRINX offer ?

FRINX UniConfig supports local authentification, password authentification, public key authentification Token authentification, RADIUS-based authentification and subtree-based authentification via AAA Shiro project.

---

## How does FRINX report problems with device interaction ?

If a device cannot be reached during a UniConfig transaction (after trying to re-establish the connection), a timeout occurs and the cause for the transaction failure is reported.

UniConfig also uses keepalive messages to continuously verify the connection to devices, using both NETCONF and CLI management protocols.

---

## Is it possible to backup a configuration ?

UniConfig stores all committed configurations for devices, templates, and snapshots in a PostgreSQL database. We suggest using existing techniques for backup that are also provided by PostgreSQL.

---

## Is it possible to enforce policies over configuration changes ?

All customer-specific validations and policy enforcements can be implemented in layers above UniConfig.

---

## Which languages are the libraries to access FRINX in?

UniConfig is written in JAVA and Kotlin, which can use data objects generated from YANG. The RESTful API (RESTCONF) can be used with a language that implements the REST client (for example, Python).

---

## Does FRINX detect if a cluster node is down on its own, or does it rely on a high-availability framework ?

A UniConfig instance is stateless, i.e., it does not persist any configuration in its datastore (PostgreSQL is used for persistence) and does not keep permanent connections (connections to devices are created on-demand in the transaction).

Because of its stateless architecture, UniConfig instances in the "cluster" do not need to communicate with each other and require no coordination. Keep in mind that requests belonging to the same transaction must be forwarded to the same UniConfig backend. For this purpose, you can use any HA component that supports sticky sessions based on cookies (for example, HA-proxy or Traefik).

---

## Can FRINX report problems to a network monitoring system ?

FRINX UniConfig can propagate NETCONF notifications and internal UniConfig notifications or data-change-events from web sockets on the Northbound API.

---

## Is additional logging available in UniConfig ?

Yes. Each component writes logs at a different verbosity level (ERROR, WARN, INFO, DEBUG, TRACE). We use the logback framework for logging messages.

Logging can be adjusted by modifying the **config/logback.xml** file. This file can be updated also on runtime. 

Another option to adjust logging for specific components is to use a logging controller. For more information, see [Logging](https://docs.frinx.io/frinx-uniconfig/user-guide/operational-procedures/logging/).

---

## Where do I find the device status and error messages when installing does not work ?

The install/uninstall process is performed automatically. The device is installed when UniConfig must read or write data from or to the device, and uninstalled at the end of the transaction if no other transaction is using the same installpoint. The install process is transparent, and the user need not care about the process outside of debugging purposes.

To get the status of the install process for all devices in the system, issue the following request. It shows the status and last connect attempt cause:

**CLI devices:**

```bash
curl --location --reQuestionuest GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=cli?content=nonconfig' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

**NETCONF devices:**

```bash
curl --location --reQuestionuest GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-netconf?content=nonconfig' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

**gNMI devices:**

```bash
curl --location --reQuestionuest GET 'http://localhost:8181/rests/data/network-topology:network-topology/topology=gnmi-topology?content=nonconfig' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

---

## What exactly happens during the installation process ?

The entire installation process includes the following:

1. Open internal transaction.
2. Install device with input parameters (CLI or NETCONF).
    1. Open IO session to device (TCP session with SSH and/or NETCONF on top of SSH session).
    2. Expose mountpoint (via internal API and RESTCONF API).
3. Sync configuration from device.
4. Write configuration and store information in database.
5. Uninstall device.
6. Commit transaction. 

---

## Why can't I install Junos devices on UniConfig ?

If installing Junos devices is not possible, UniConfig may give this response :

```
2020-03-23 03:26:07,174 ERROR [default-pool-5-2] (org.opendaylight.netconf.sal.connect.netconf.NetconfDevice) - RemoteDevice{junos}: Initialization in sal failed, disconnecting from device
java.lang.IllegalStateException: Cannot build Netconf device schema context.
at org.opendaylight.netconf.cache.impl.utils.NetconfSchemaContextFactory.buildSchemaContext(NetconfSchemaContextFactory.javanswer:64)
at org.opendaylight.netconf.cache.model.SchemaRepositoryRefs.buildSchemaContext(SchemaRepositoryRefs.javanswer:68)
at org.opendaylight.netconf.sal.connect.netconf.NetconfDevice$SchemaSetup.run(NetconfDevice.javanswer:515)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.javanswer:511)
at com.google.common.util.concurrent.TrustedListenableFutureTask$TrustedFutureInterruptibleTask.runInterruptibly(TrustedListenableFutureTask.javanswer:125)
at com.google.common.util.concurrent.InterruptibleTask.run(InterruptibleTask.javanswer:57)
at com.google.common.util.concurrent.TrustedListenableFutureTask.run(TrustedListenableFutureTask.javanswer:78)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.javanswer:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.javanswer:624)
at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.javanswer:30)
at java.lang.Thread.run(Thread.javanswer:748)
Caused by: java.lang.IllegalStateException: There are not any sources left for building of schema context.
at org.opendaylight.netconf.cache.impl.utils.NetconfSchemaContextFactory.setUpSchema(NetconfSchemaContextFactory.javanswer:114)
at org.opendaylight.netconf.cache.impl.utils.NetconfSchemaContextFactory.buildSchemaContext(NetconfSchemaContextFactory.javanswer:61)
... 10 more
```

To install Junos devices, set up a NetConf session that is compliant with RFC and Yang schemas (`rfc-compliant` and `yang-compliant`) on the Junos device:

```
> configure
# set system services netconf rfc-compliant
# set system services netconf yang-compliant
# commit
```