<!-- Release notes generated using configuration in .github/release.yml at 5.2.4 -->

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
### ✅ New Features
* [UNIC-1486] [UNIC-1509] Skip yang constraints 
* [UNIC-1507] : One uniconfig tx for (un)install-multiple-nodes RPC
### 💡 Improvements
* [UNIC-1514] Add support CER device version 12.*
* [DEP-616] UDP loadbalancer with ip transparency for SNMP notifications
* [UNIC-1407] Adjust LOGs in Uniconfig shell
* [UNIC-1438] Refactoring sal-fallback-provider / fallback-schema-service
* [UNIC-1495] Refactor mountpoint to JOOQ style
* Add new Release workflow
* [UNIC-1454] Bump revision date in updated logging YANG models.
* Remove object cache
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
