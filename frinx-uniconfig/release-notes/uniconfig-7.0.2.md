<!-- Release notes generated using configuration in .github/release.yml at 7.0.2 -->

## What's Changed
### 🐞 Bug Fixes
* Fix close-transaction rpc called while another rpc is running
* Fix tx close on shell
* [UNIC-1853] Fix restconf logging rpcs
* [UNIC-1859] Fix netconf health-check connection leak
### 💡 Improvements
* Refactor file name pattern for log files.
* [UNIC-1852] Add pipe commands for all show operation.
* [UNIC-1846] Implement is-in-sync rpc multinode version.
* [UNIC-1856] Add calc-diff and calc-git-like-diff unit tests in java client.
* [UNIC-1856] Multinode sync from network rpc java client
* [UNIC-1856] Multinode sync to network rpc java client
* [UNIC-1857] Implement addtional MIB grammar.
* [UNIC-1854] OpenApi - Adjust Install Node RPC to be valid without changes
### 🔨 Dependency Upgrades
* build(deps): bump grpc.version from 1.66.0 to 1.68.0
* build(deps): bump org.jetbrains:annotations from 24.1.0 to 25.0.0
* build(deps): bump com.google.guava:guava from 33.3.0-jre to 33.3.1-jre
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.23 to 2.2.24
* build(deps): bump jakarta.inject:jakarta.inject-api from 2.0.1 to 2.0.1.MR
* build(deps): bump io.swagger.core.v3:swagger-core from 2.2.24 to 2.2.25
* build(deps): bump org.apache.maven.plugins:maven-failsafe-plugin from 3.5.0 to 3.5.1
* build(deps): bump org.checkerframework:checker-qual from 3.47.0 to 3.48.0
* build(deps): bump org.apache.maven.plugins:maven-surefire-plugin from 3.5.0 to 3.5.1
* build(deps): bump com.google.errorprone:error_prone_core from 2.32.0 to 2.33.0
* build(deps): bump org.ow2.asm:asm-bom from 9.7 to 9.7.1
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.18.1 to 10.18.2
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 2.0.20 to 2.0.21
* build(deps): bump org.checkerframework:checker-qual from 3.48.0 to 3.48.1
* build(deps): bump org.apache.maven.plugins:maven-javadoc-plugin from 3.10.0 to 3.10.1
* build(deps): bump org.jetbrains:annotations from 25.0.0 to 26.0.0
* build(deps): bump com.google.errorprone:error_prone_annotations from 2.32.0 to 2.33.0
* build(deps): bump sshd.version from 2.13.2 to 2.14.0
* build(deps): bump actions/upload-artifact from 4.4.0 to 4.4.3
### 🔧 Other Changes
* Bump to 7.0.2-SNAPSHOT
* Add CLI install support for juniper 23.* (#2754)
* [UNIC-1834] Improve modiffication diff
* Update YourKit profiler agent to 2024.9
* Update YourKit version in run_uniconfig.sh
* Remove FIXME, no integration tests rely on this log message.
* Fix close-tx issues
* Release 7.0.2.
