<!-- Release notes generated using configuration in .github/release.yml at 6.1.1 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1739] - Fix advisory session unlocking
* Swagger: Fix missing cli topology in path parameters
* Swagger: Fix missing snmp topology in path parameters
* [UNIC-1753] Multiple gnmi notification fixes for IOS Xr7
* [UNIC-1751] Fix mount operation duplication in the same tx
* [UNIC-1756] Fix wrong snmp connection establishment
* [UNIC-1754] Fix yang-patch rfc8040 error response
* Fixed JUNOS version 13 installation (#2485)
* [UNIC-1758] Database error: Failed to update node YANG repository because repository does not exist in DB
* Fix augmentation qname handling
* [UNIC-883] Fix DB tx exception handling
* Fix cli session closeup
* Fix traefik changing URL query param from ; to &
* Support check-nodes-connection for southbound only
* [UNIC-1763] Swagger: Fix generation of action nodes and tags
* Fix default reading properties
* [UNIC-1771] Fix register-repository
* Fix 'End of input' exception on netconf session.close()
### ✅ New Features
* [UNIC-1580] - Integration of 'fields' query parameter into NETCONF subtree filtering
* [UNIC-1762] Bulk-get RPC
### 💡 Improvements
* [UNIC-1743] Create calculate-diff-shell rpc.
* Add banner to logs
* [UNIC-1761] Fix time range in gnmi streaming
* [UNIC-1400] Refactor and extract uniconfig diff
* Package logback cluster configuration
* Swagger - add module name to choice schema node definition
### 🔨 Dependency Upgrades
* build(deps): bump org.apache.maven.plugins:maven-shade-plugin from 3.5.3 to 3.6.0
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.13.0 to 3.13.1
* build(deps): bump org.apache.maven.plugins:maven-enforcer-plugin from 3.4.1 to 3.5.0
* build(deps): bump org.apache.maven.plugins:maven-javadoc-plugin from 3.6.3 to 3.7.0
* build(deps): bump com.google.errorprone:error_prone_core from 2.27.1 to 2.28.0
* build(deps): bump com.google.errorprone:error_prone_annotations from 2.27.1 to 2.28.0
* build(deps): bump com.google.guava:guava from 33.2.0-jre to 33.2.1-jre
* build(deps): bump docker/login-action from 3.1.0 to 3.2.0
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.13.0 to 3.13.1
* build(deps): bump org.apache.maven.plugins:maven-checkstyle-plugin from 3.3.1 to 3.4.0
* build(deps): bump commons-net:commons-net from 3.10.0 to 3.11.0
* build(deps): bump org.checkerframework:checker-qual from 3.43.0 to 3.44.0
* build(deps): bump io.github.git-commit-id:git-commit-id-maven-plugin from 8.0.2 to 9.0.0
* build(deps): bump org.apache.maven.plugins:maven-surefire-plugin from 3.2.5 to 3.3.0
* build(deps): bump org.apache.maven.plugins:maven-failsafe-plugin from 3.2.5 to 3.3.0
* build(deps): bump org.apache.maven.plugins:maven-dependency-plugin from 3.6.1 to 3.7.0
* build(deps): bump org.apache.maven.plugins:maven-release-plugin from 3.0.1 to 3.1.0
* build(deps): bump org.apache.maven.plugins:maven-project-info-reports-plugin from 3.5.0 to 3.6.0
* build(deps): bump commons-net:commons-net from 3.11.0 to 3.11.1
* build(deps): bump docker/build-push-action from 5 to 6
* build(deps): bump maven.core.version from 3.9.7 to 3.9.8
* build(deps): bump org.apache.maven.plugins:maven-jar-plugin from 3.4.1 to 3.4.2
* build(deps): bump org.apache.maven.plugins:maven-clean-plugin from 3.3.2 to 3.4.0
* build(deps): bump com.github.spotbugs:spotbugs-annotations from 4.8.5 to 4.8.6
* build(deps): bump jline.version from 3.26.1 to 3.26.2
* build(deps): bump org.apache.maven.plugins:maven-dependency-plugin from 3.7.0 to 3.7.1
### 🔧 Other Changes
* Bump to 6.1.1-SNAPSHOT.
* [UNIC-1764] Support to optionally wrap cases in choice node
* Release 6.1.1.
