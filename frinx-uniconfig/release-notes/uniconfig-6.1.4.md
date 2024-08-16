<!-- Release notes generated using configuration in .github/release.yml at 6.1.4 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1801] Fix health-check clash with un/mounting process
* [UNIC-1804] Fix transaction timestamp
* [UNIC-1813] Fix replace-config-with-snapshot
* [UNIC-1808] Commit fail on metadata update
* [UNIC-1811] Fix loading yang repository in bulk-get cluster
* [UNIC-1814] Fix template auto-upgrade (#2648)
* Change log level in GlobalDOMSchemaServiceImpl
### 💡 Improvements
* [UNIC-1786] Open prometheus endpoint in actuator.
* [UNIC-1786] Rewrite UniConfig metrics to micrometer.
* [UNIC-1806] TopologyNode based data-broker
* [UNIC-1812] Implement casa init unit.
* [UNIC-1803] Local Uniconfig deployment in minikube(kubernets cluster)
### 🔨 Dependency Upgrades
* build(deps): bump antlr4.version from 4.13.1 to 4.13.2
* build(deps): bump sshd.version from 2.13.1 to 2.13.2
* build(deps): bump org.checkerframework:checker-qual from 3.45.0 to 3.46.0
* build(deps): bump actions/upload-artifact from 4.3.4 to 4.3.5
* build(deps): bump com.google.errorprone:error_prone_annotations from 2.29.2 to 2.30.0
* build(deps): bump actions/upload-artifact from 4.3.5 to 4.3.6
* build(deps): bump org.apache.commons:commons-compress from 1.26.2 to 1.27.0
* build(deps): bump org.codehaus.mojo:exec-maven-plugin from 3.3.0 to 3.4.0
* build(deps): bump com.google.errorprone:error_prone_core from 2.29.2 to 2.30.0
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 2.0.0 to 2.0.10
* build(deps): bump io.zonky.test.postgres:embedded-postgres-binaries-linux-amd64 from 16.3.0 to 16.4.0
* build(deps): bump grpc.version from 1.65.1 to 1.66.0
### 🔧 Other Changes
* Bump to 6.1.4-SNAPSHOT.
* Readme update regarding topology-node data brokers
* Release 6.1.4.
