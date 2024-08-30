<!-- Release notes generated using configuration in .github/release.yml at 6.1.5 -->

## What's Changed
### 🐞 Bug Fixes
* [UNIC-1823] Add path to error for invalid value
* Fix show_uniconfig_status.sh
* [UNIC-1618] Attempt to install vnf20 device (10.103.5.202) with snmp v3 fail
* [UNIC-1817] Fix synchronization of collecting nested tx db writers
* [UNIC-961] Option to add tag to leaf without providing value
### 💡 Improvements
* [UNIC-1805] Rewrite Uniconfig healthcheck using Spring Boot actuator.
* [UNIC-1732] Asynchronous apply-template RPC
* Clean up code related to old health rpc impl.
* Fix error exposing internal info
* Move checking db into readiness health indicator.
* Reimplementing rpc health with spring healthcheck logic
### 🔨 Dependency Upgrades
* build(deps): bump commons-cli:commons-cli from 1.8.0 to 1.9.0
* build(deps): bump com.google.guava:guava from 33.2.1-jre to 33.3.0-jre
* build(deps): bump org.apache.maven.plugins:maven-surefire-plugin from 3.3.1 to 3.4.0
* build(deps): bump org.apache.maven.plugins:maven-failsafe-plugin from 3.3.1 to 3.4.0
* build(deps): bump org.codehaus.mojo:exec-maven-plugin from 3.4.0 to 3.4.1
* build(deps): bump org.apache.maven.plugins:maven-plugin-plugin from 3.13.1 to 3.14.0
* build(deps-dev): bump org.apache.maven.plugin-tools:maven-plugin-annotations from 3.13.1 to 3.14.0
* build(deps): bump maven.core.version from 3.9.8 to 3.9.9
* build(deps): bump org.apache.maven.plugins:maven-project-info-reports-plugin from 3.6.2 to 3.7.0
* build(deps): bump com.puppycrawl.tools:checkstyle from 10.17.0 to 10.18.0
* build(deps): bump org.apache.maven.plugins:maven-checkstyle-plugin from 3.4.0 to 3.5.0
* build(deps): bump org.apache.maven.plugins:maven-invoker-plugin from 3.7.0 to 3.8.0
* build(deps): bump org.apache.maven.plugins:maven-dependency-plugin from 3.7.1 to 3.8.0
* build(deps): bump org.apache.commons:commons-compress from 1.27.0 to 1.27.1
* build(deps): bump org.jetbrains.kotlin:kotlin-maven-plugin from 2.0.10 to 2.0.20
### 🔧 Other Changes
* Bump to 6.1.5-SNAPSHOT.
* [UNIC-1820] - Fix local-as command for IOS device
* Release 6.1.5.

## New Contributors
