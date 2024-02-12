<!-- Release notes generated using configuration in .github/release.yml at 5.2.5 -->

## What's Changed
### 🐞 Bug Fixes
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
### ✅ New Features
* [UNIC-1533] - skip length & range constraints
* [UNIC-1533] - skip length & range constraints
* [UNIC-1383] Create logging broker for GNMI notifications
* Support tailf's comment and label audit params in netconf southbound
### 💡 Improvements
* [UNIC-1477] Unify HTTP clients
* Refactor TaskExecutor
* [UNIC-1505] - Parallel syncFromNetwork
* [UNIC-963] Convert List<ImmutablePair> to Map in GNMI
* [UNIC-1102] Improve logging for schema context building
* [UNIC-1541] Refactor DCE to JOOQ style
* [UNIC-1472] CLI shell - Support multiple list entries
* [UNIC-1549] Refactor netconf-key-credential to JOOQ style
### 🔨 Dependency Upgrades
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
