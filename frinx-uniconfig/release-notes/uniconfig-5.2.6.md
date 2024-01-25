<!-- Release notes generated using configuration in .github/release.yml at 5.2.6 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1517] Fix pagination
* Fix getting fallback schema token.
* [UNIC-1453] UniConfig Shell - one-liner show/set/delete for callbacks
* Fix UC not start
* [UNIC-1422] Fixed transaction is closed issue
* Fix failing cli shell tests
* [UNIC-1521] Fix reading list nodes in augmentations (#1946)
* Fix SNMP v2c notifications
* [UNIC-1512] Fix ordering of operations on gnmi topology
* Add handling for successful rpc with status code 204.
* Cover more errors in connect node RPC responses
* Add handling for successful rpc with status code 204.
* [UNIC-1526] - increase gnmi executor queue size
* Fix callbacks
* Fix sync empty target-nodes response
* [UNIC-1538] Fixed check-installed-nodes RPC
* FR-63 Fix parsing of SROS CLI prompt in the transaction mode
* [UNIC-1534] Swagger: fix bad types for numeric attributes
* Fixed command for disabling pagination on SROS
* [UNIC-1539] Fix some typos in descriptions
* [UNIC-1537] - fix (un)install-multiple
* Fix CLI Shell command length counter
* [UNIC-1547] Fix sync-from-network RPC
* [UNIC-1557] Fix MountPointListenerException
* [UNIC-1550] - mount/unmount synchronization
* [UNIC-1527] - fixing show mode bugs
* add missing docker dependency
* [UNIC-1573] CLI shell: Fix Connect/Disconnect RPC suggestions
* [UNIC-1576] - Fix retrieving of augmentation node
* [UNIC-1574] Fix remove patch operation
* [UNIC-1481] Fix with-defaults query parameter
### ✅ New Features
* [UNIC-1486] [UNIC-1509] Skip yang constraints 
* [UNIC-1507] : One uniconfig tx for (un)install-multiple-nodes RPC
* [UNIC-1533] - skip length & range constraints
* [UNIC-1533] - skip length & range constraints
* [UNIC-1383] Create logging broker for GNMI notifications
* Support tailf's comment and label audit params in netconf southbound
* [UNIC-1545] - Sync southbound nodes
### 💡 Improvements
* [UNIC-1514] Add support CER device version 12.*
* [DEP-616] UDP loadbalancer with ip transparency for SNMP notifications
* [UNIC-1407] Adjust LOGs in Uniconfig shell
* [UNIC-1438] Refactoring sal-fallback-provider / fallback-schema-service
* [UNIC-1495] Refactor mountpoint to JOOQ style
* Add new Release workflow
* [UNIC-1454] Bump revision date in updated logging YANG models.
* Remove object cache
* [UNIC-1477] Unify HTTP clients
* Refactor TaskExecutor
* [UNIC-1505] - Parallel syncFromNetwork
* [UNIC-963] Convert List<ImmutablePair> to Map in GNMI
* [UNIC-1102] Improve logging for schema context building
* [UNIC-1541] Refactor DCE to JOOQ style
* [UNIC-1472] CLI shell - Support multiple list entries
* [UNIC-1549] Refactor netconf-key-credential to JOOQ style
* [UNIC-1499] Swagger: disable generation of POST list entries
* [UNIC-1551] Refactor Snapshot and SnapshotNode to JOOQ
* [UNIC-1567] - unify error-info property
### 🖥️ API Changes
* [UNIC-1288] Refactor logging and restconf-logging RPCs
* Fix generic status code to specific status code for Subtree RPCs.
### 🔨 Dependency Upgrades
* build(deps): bump com.github.spotbugs:spotbugs-annotations from 4.8.0 to 4.8.1
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.7.3.6 to 4.8.1.0
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.18 to 2.2.19
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.10.1 to 3.10.2
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.10.1 to 3.10.2
* build(deps): bump io.zonky.test.postgres:embedded-postgres-binaries-linux-amd64 from 13.12.0 to 13.13.0
* build(deps): bump org.jetbrains:annotations from 24.0.1 to 24.1.0
* build(deps): bump bouncycastle.version from 1.76 to 1.77
* build(deps): bump org.codehaus.mojo:exec-maven-plugin from 3.1.0 to 3.1.1
* build(deps): bump protobuf.version from 3.25.0 to 3.25.1
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.12.4 to 10.12.5
* build(deps): bump org.owasp:dependency-check-maven from 8.4.2 to 8.4.3
* build(deps): bump org.apache.commons:commons-compress from 1.24.0 to 1.25.0
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 1.9.20 to 1.9.21
* build(deps): bump com.fasterxml.jackson.core:jackson-databind from 2.15.3 to 2.16.0
* build(deps): bump org.apache.maven.plugins:maven-project-info-reports-plugin from 3.4.5 to 3.5.0
* build(deps): bump io.zonky.test:embedded-postgres from 2.0.4 to 2.0.5
* Use latest maven plugins
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.8.1.0 to 4.8.2.0
* build(deps): bump com.github.spotbugs:spotbugs-annotations from 4.8.1 to 4.8.2
* build(deps): bump grpc.version from 1.59.0 to 1.59.1
* build(deps): bump commons-io:commons-io from 2.15.0 to 2.15.1
* build(deps): bump maven.core.version from 3.9.5 to 3.9.6
* build(deps): bump actions/setup-java from 3 to 4
* Bump Spring Boot to 3.1.6
* build(deps): bump org.apache.maven.plugins:maven-javadoc-plugin from 3.6.2 to 3.6.3
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.12.5 to 10.12.6
* build(deps): bump io.zonky.test.postgres:embedded-postgres-binaries-linux-amd64 from 13.13.0 to 13.13.1
* build(deps): bump grpc.version from 1.59.1 to 1.60.0
* build(deps): bump org.codehaus.mojo:build-helper-maven-plugin from 3.4.0 to 3.5.0
* build(deps): bump actions/setup-python from 4.7.1 to 5.0.0
* build(deps): bump io.zonky.test:embedded-postgres from 2.0.5 to 2.0.6
* build(deps): bump org.owasp:dependency-check-maven from 8.4.3 to 9.0.4
* Bump org.springframework.boot from 3.1.6 to 3.1.7
### 🔧 Other Changes
* Release new snapshot version 5.2.4-SNAPSHOT
* Workflows: enable immutable and mutable cluster tests.
* Workflows: add new test to embeded workflow - skip_yang_constraints.
* Workflows: fix typo for embeded tests.
* Fixed logging in TaskExecutorImpl
* Add name to release workflow
* Allow double quotes in release notes body
* Use PAT in release workflow
* Workflows: fix typo in embededDB_cross_versions workflow.
* Fix PAT usage
* Dispatch on wf completion instead
* Workflows: add cache for embeded test and remove all docker images
* Merge publish release notes into release wf as a separate job
* Fix release workflow
* Update release.yml
* [UNIC-1502] SNMP v3 notifications
* Fix SNMP bean creation
* README - Update release process
* Add distributionSuffix property
* Build distro without units on release
* Workflows: disable delete_customers_license_server.yml
* Forgot to remove these
* Forgot to remove this one as well
* Release 5.2.4.
* Rename UniConfig distribution name from bin to without TU
* Bump to 5.2.5-SNAPSHOT.
* Node jooq
* [UNIC-1531] Use Java 17 as a target
* Cluster notification
* Revert "build(deps): bump com.fasterxml.jackson.core:jackson-databind from 2.15.3 to 2.16.0 (#1962)"
* Fixed usage of JOOQ enumerations
* fix_sync_from_network_tests
* Revert "[UNIC-1533] - skip length & range constraints"
* Suppress logback
* [UNIC-1523] Refactor notifications and subscriptions to JOOQ style
* [UNIC-1542] Refactor Session to JOOQ style
* [UNIC-1539] Fix some typos
* Fixed creation of NormalizedNode from read NetconfKeyCredential
* Revert "[UNIC-1547] Fix sync-from-network RPC (#2043)"
* Revert "Revert "[UNIC-1547] Fix sync-from-network RPC (#2043)" (#2049)"
* Suppress sshd vulnerability
* CVE-2023-51074 suppress
* Release 5.2.5.
* Bump to 5.2.6-SNAPSHOT.
* Change default value for connect-node RPC
* [UNIC-1581] Fix client update and read properties structure
* Release 5.2.6.
