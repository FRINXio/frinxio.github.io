<!-- Release notes generated using configuration in .github/release.yml at 5.2.1 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1395] Fix key delimiter in URI
* [UNIC-1390] Swagger: Fix RPCs placed after mountpoint
* [UNIC-1273] Use Jetty embedded server
* [UNIC-1399] Fix issues with shell
* [UNIC-1399] Switch shell terminal back to JNA
* [UNIC-1410] Fix tx cleanup when request fails
* [UNIC-1410] Fix_tx_closing
* [UNIC-1413] Fixed updating snapshot in immediate-commit model
* [UNIC-1420] Fix cli ssh session reconnect
* [UNIC-1425] Fix crypto bug
* [UNIC-1352, UNIC-1254] Fix cluster issues
* Cleanup test resources properly
* [UNIC-1340] Fixed releasing of used YANG modules from memory
* [UNIC-1432] Swagger: Fix generation of post list endpoints
* [UNIC-1429] Fix replace is sent using delete operation
* [UNIC-1423] Fix identityRef as listEntry key in templatesg
* Prevented sending no description command if there is no change for rpd description
* Fix show_uniconfig_status script.
* [UNIC-1446] Fix SpotBugs violations - Reliance on default encoding
* [UNIC-1404] UniConfig Shell - fix system augmentation
* [UNIC-1430] Fix replace yang-patch for gnmi mountpoint
* [UNIC-1448] Fix SpotBugs violations - Use a localized version of String.toUpperCase() and String.toLowerCase()
* Fix mapEntryNodes in gnmi notifications
* Fix calculate diff rpc
* SNMP Node id is incorrectly parsed
* [UNIC-1447] Fix SpotBugs violations - Multithreaded correctness
* Additional fix to calculate diff rpc
* [UNIC-1463] Remove duplicates from Set
* Fix skip of unreachable-nodes.
* SNMP adjust exception
* Fix reading of actual YANG repository from mountpoint data
* Fixed DateTime format in the transaction-log
* Fix immediate commit model and submit successfull nodes.
* Add git registry to dependabot.yml
* Registry attempt no.2
* Fix show UC status script
* Fix SNMP Notification bean creation
* [UNIC-1451] Fix SpotBugs violations - Correctness
* [UNIC-1365] Gnmi stream fixes
* Data-change-events publisher fix
* Fixed loading of YANG from path in client diff tool (#1747)
### ✅ New Features
* [UNIC-1394] Client side diff
* [UNIC-1373] Implemented dryrun-commit for GNMI topology
* [UNIC-1398] SNMP notifications
* [UNIC-1402] UC Shell - default callbacks repository
* [UNIC-1218] Add dynamic property module
### 💡 Improvements
* [UNIC-1386] Map correct ObjectTypes to NotificationTypes
* [UNIC-1391] Add SNMP notifications to SchemaContext
* [UNIC-1258] Fix issues reported by SpotBugs
* [UNIC-980] UniconfigShell: Improving suggestions menu
* [UNIC-1223] ODL parent cleanup
* [UNIC-1273] Adjust bootstrapping of web containers
* [UNIC-1242] Unify generation of release notes
* [UNIC-1218] Rewrite database connection pool and connection properties
* [UNIC-981] UniconfigShell: Remove explicit show submode from root mode
* [UNIC-1370] Adjust notification result parsing
* [UNIC-1242] Remove PR links from the uploaded release notes
* [UNIC-1411] Add mapping to request/response log message (#1630)
* [UNIC-1412] change gnmi packaging
* [UNIC-1394] Add overloaded build methods to client side diff
* [UNIC-1394] Remove gnmi dependencies from java client
* [UNIC-1154] Integrate JOOQ into database access layer
* [UNIC-1401] UniConfig Shell - one line SET / DELETE command
* [UNIC-1403] Unified format of shell audit logs
* [UNIC-1245] Replacing Guava future by CompletableFuture
* [PANT-83] add logs for pant83
* [UNIC-1441] SNMP config classes
* [UNIC-1435] Refactor transaction-log to JOOQ style
* [UNIC-1463] Fix SpotBugs violations - Code vulnerabilities
* [UNIC-1449] Fix SpotBugs violations - Performance
### 🖥️ API Changes
* [UNIC-1289] Refactor RPCs: revert-changes, query-config, device-discovery
* [UNIC-1380] Add gnmi-messages logging broker
* [UNIC-1287] Refactor snapshot-manager RPCs
* [UNIC-1282] Refactoring uniconfig manager RPCs
### 🔨 Dependency Upgrades
* build(deps): bump grpc.version from 1.57.1 to 1.57.2
* build(deps): bump protobuf.version from 3.23.4 to 3.24.0
* build(deps): bump io.zonky.test.postgres:embedded-postgres-binaries-linux-amd64 from 13.11.0 to 13.12.0
* [UNIC-1223] Align an embedded kafka version with the clients provided by Spring
* build(deps): bump protobuf.version from 3.24.0 to 3.24.1
* build(deps): bump org.owasp:dependency-check-maven from 8.3.1 to 8.4.0
* build(deps): bump com.github.gantsign.maven:ktlint-maven-plugin from 1.16.0 to 2.0.0
* build(deps): bump spring.boot.version from 3.1.2 to 3.1.3
* build(deps): bump org.apache.maven.plugins:maven-enforcer-plugin from 3.3.0 to 3.4.0
* build(deps): bump protobuf.version from 3.24.1 to 3.24.2
* build(deps): bump kotlin.version from 1.9.0 to 1.9.10
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.12.2 to 10.12.3
* build(deps): bump org.apache.maven.plugins:maven-javadoc-plugin from 3.5.0 to 3.6.0
* build(deps): bump org.apache.commons:commons-compress from 1.23.0 to 1.24.0
* build(deps): bump grpc.version from 1.57.2 to 1.58.0
* build(deps): bump antlr4.version from 4.13.0 to 4.13.1
* build(deps): bump protobuf.version from 3.24.2 to 3.24.3
* build(deps): bump org.apache.maven.plugins:maven-enforcer-plugin from 3.4.0 to 3.4.1
* build(deps): bump spring.boot.version from 3.1.3 to 3.1.4
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.15 to 2.2.16
* build(deps): bump com.github.spotbugs:spotbugs-maven-plugin from 4.7.3.5 to 4.7.3.6
* build(deps): bump org.apache.maven.plugins:maven-shade-plugin from 3.5.0 to 3.5.1
* build(deps): bump org.jetbrains.dokka:dokka-maven-plugin from 1.8.20 to 1.9.0
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.12.3 to 10.12.4
* build(deps): bump commons-io:commons-io from 2.13.0 to 2.14.0
* build(deps): bump com.github.gantsign.maven:ktlint-maven-plugin from 2.0.0 to 3.0.0
* Maven 3.9.5
* build(deps): bump commons-net:commons-net from 3.9.0 to 3.10.0
* build(deps): bump protobuf.version from 3.24.3 to 3.24.4
* build(deps): bump org.immutables:value from 2.9.3 to 2.10.0
* build(deps): bump org.springframework.cloud:spring-cloud-dependencies from 2022.0.3 to 2022.0.4
* build(deps): bump actions/checkout from 3 to 4
* build(deps): bump actions/upload-artifact from 3.1.2 to 3.1.3
* build(deps): bump docker/build-push-action from 4 to 5
* build(deps): bump actions/setup-python from 4.6.1 to 4.7.1
* build(deps): bump docker/login-action from 2.2.0 to 3.0.0
* build(deps): bump org.json:json from 20230618 to 20231013
### 🔧 Other Changes
* 5.2.1-SNAPSHOT
* [FI-1693] Remove Jenkins-test from merge workflow
* Workflows: update cluster IP from 10.19.0.67 to 10.19.0.242 and
* Workflows: remove VPN to FRINX for postgresDB.
* Workflows: update path to new VM of postgresDB.
* Workflows: update path and remove FRINX VPN for embeded tests.
* Removed forgotten LOG
* Release 5.2.1
