# UniConfig Client (SDK)

Uniconfig client SDK is implemented in Java 17 and uses Uniconfig's RESTconf API.

The SDK provides following advantages over raw RESTconf:

- SDK is versioned and tied tied to a specific Uniconfig release
- Every version is tested
- Type safety
- Additional features on top of basic RESTconf facade such as: 
  - Integration with Kafka (notification listener)
  - Client side diff (to calculate diff between 2 versions of config)
  - etc.

## Basic device configuration management

An example of a simple read & write use case on top of a single device called `vnf`.

```java
 try (ReadWriteUniconfigTx rwTx = uniconfig.readWriteTransaction()) {
            final Device vnf = rwTx.device(DEVICE_ID);

            final IsInSyncStatus inSyncStatus = vnf.isInSync();
            LOG.info("Is in sync: {}", inSyncStatus.getInSync());

            if (!inSyncStatus.getInSync()) {
                final SyncConfigFromNetworkStatus syncConfigFromNetworkStatus = vnf.syncConfigFromNetwork();
                LOG.info("Sync from network output: {}", syncConfigFromNetworkStatus);
            }

            // read user with id=foo
            final Optional<ListEntry> readEntry = vnf.config().containerPath("system", "system")
                    .list("users")
                    .listEntryPath(LeafNode.of("name").setStringValue("foo"))
                    .read();
            if (!readEntry.isPresent()) {
                LOG.info("Data node on path 'system:system/users=foo' doesn't exist");
            }

            // delete description under interface 'tvi-0/3'
            final LeafNodePath descriptionLeaf = vnf.config()
                    .containerPath("interfaces", "interfaces")
                    .list("t")
                    .listEntryPath(LeafNode.of("name").setStringValue("t-0/3"))
                    .leaf("description")
                    .delete();

            // change leaf value: set 'enable' flag
            vnf.config()
                    .containerPath("interfaces", "interfaces")
                    .list("t")
                    .listEntryPath(LeafNode.of("name").setStringValue("t-0/2"))
                    .leaf("enable")
                    .createDataNode()
                    .setBooleanValue(false)
                    .put();

            final UniconfigResponse<CommitStatus> commitOutput = rwTx.commit();
            LOG.info("Commit output: {}", commitOutput);
            if (!commitOutput.isOperationSuccessful()) {
                throw new IllegalStateException("Commit failed with status: " + commitOutput);
            }
 }
```

## Integration with Kafka

Uniconfig SDK implements a kafka listener that taps into Uniconfig notifications streams and allows the client to consume the notifications easily. Notifications available through Uniconfig are: Device notifications, alerts and telemetry but also Uniconfig generated notifications in the audit log topic. For further information see [Uniconfig notifications (kafka)](frinx-uniconfig/user-guide/uniconfig-operations/kafka-notifications/readme.md).

## Client side diff

Uniconfig SDK offers a diff calculation feature that can calculate a delta between a `before` device configuration state and `after` configuration state. This diff is then transformed into a [patch operation](https://www.rfc-editor.org/rfc/rfc8072.html) and sent to Uniconfig's RESTconf API.


Client side diff calculation is useful in specific cases, where a substential amount of configuration data is avalable and modified outside of Uniconfig. Typically, the entire `after` state would have to be pushed into Uniconfig and Uniconfig would calculate its own diff internally. By calculating on the client side, it is possilble to reduce the network communication between client and Uniconfig (as less data has to be sent) and to reduce how much diff calculation Uniconfig has to do itself (shifting it to the client side).


This feature leverages the same implementation of diff calculation that is used withing Uniconfig itself and requires YANG schemas to be present for the computation.

Example usage:

```java
try(var afterStream = new FileInputStream(after.toFile());
                var beforeStream = new FileInputStream(before.toFile());
                var cfg = vnf.config()) {

                // Push before config to make sure UC's intent contains before data
                LOG.info("Pushing before config: {}", DEVICE_ID);
                cfg.patchEntireConfig("{ \"configuration\":" + Files.readString(before) + "}");

                LOG.info("Calculating diff {}", DEVICE_ID);
                final JSONObject diff = Diff.build(
                                schemas, beforeStream, afterStream,
                                new Diff.DiffSettings(
                                        false, // if true, ignore unknown json elements (not in YANG)
                                        Set.of() // set of extensions in YANG to ignore marked data e.g. deprecated
                                ))
                        .toPatch("client sample");

                LOG.info("Done calculating diff {}", DEVICE_ID);
                LOG.info(diff.toString(2));
                LOG.info("Sending diff to Uniconfig {}", DEVICE_ID);
                cfg.yangPatchEntireConfig(diff);

                // close the transaction
            } catch (DiffException e) {
                LOG.error("CLient side diff failed {}", DEVICE_ID, e);
                // Diff failed
                throw e;
            }
}
```

Notable features of client side diff:
- Requires YANG schemas in order to work
- JSON data have to conform to those YANG models
- Allows for templated values to be present in the JSON
- Produces a single PATCH operation containing all changes detected. See [YANG patch in Uniconfig](/frinx-uniconfig/user-guide/uniconfig-operations/yang-patch/readme.md)
- Changes detected: Create, Update, Delete, Reorder (lists and leaf-lists)
- Uses the same implementation as Uniconfig's calculate-diff